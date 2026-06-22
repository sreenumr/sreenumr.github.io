---
title: "MusicMentor: Pairing .NET with a Python AI Microservice for Music Theory"
date: 2026-06-22
categories: [Engineering, .NET, Python, AI]
---

MusicMentor is an AI-powered music learning app — key detection, scales, chord progressions, transposition, and natural-language music theory Q&A. The interesting part isn't any single feature, it's the architecture: **ASP.NET Core + Blazor on top, a Python FastAPI service underneath for anything AI-shaped.**

---

## Why Split the Stack at All

Music theory is deterministic. A C major scale is always C D E F G A B — there's no reason to ask a language model for that. So the design draws a hard line:

- **C# (`MusicMentor.Core`)** owns scales, chords, key detection, progressions — pure, fast, testable, always correct.
- **Python (FastAPI)** owns natural language and AI-generated responses — the thing C# can't do.

The two talk over plain HTTP. Neither knows the other's internals:

```
.NET app                         Python app
Knows: "there is a service       Knows: "I receive POST requests
at http://localhost:8000         at /analyze and return JSON"
that I POST prompts to"
```

I actually started by adding a `/detect-key` endpoint to the Python side too — then removed it. Having both C# and the LLM detect keys means two implementations that can disagree. The AI layer should only handle what the deterministic layer can't.

---

## The Microservice Boundary, Concretely

On the .NET side, an `IAiMusicService` interface is all the controller ever sees:

```csharp
builder.Services.AddHttpClient();
builder.Services.AddScoped<IAiMusicService, AiMusicService>();
```

`AiMusicService` POSTs to a URL that lives entirely in `appsettings.json`:

```
Dev laptop:  "AiService:BaseUrl": "http://localhost:8000"   (Ollama + phi4-mini)
Desktop:     "AiService:BaseUrl": "http://192.168.1.50:8000" (MOSS-Music-8B, GPU)
```

Swapping the model underneath — phi4-mini on a CPU laptop during dev, MOSS-Music-8B at 4-bit quantization on a GTX 1070 Ti for real inference — is a one-line config change. The controller, the interface, the Blazor UI: none of it changes.

That's the actual payoff of the interface + config-driven URL combo, not the buzzwords: a junior engineer with a CPU-only laptop can develop the entire app, and a separate, more powerful machine can run the real model, without anyone touching application code.

---

## Blazor Instead of React

The whole app — API, business logic, and UI — is C#. No TypeScript duplicating C# model shapes, no separate build tooling. I went with **Blazor Web App, Interactive Server render mode**: components run server-side, a SignalR WebSocket pushes UI updates after the initial page load.

A few sharp edges showed up fast, all from treating Razor like plain HTML/C#:

- **`@bind` on a button does nothing.** `@bind` is for inputs; clicks need `@onclick`. The button rendered fine and silently did nothing — no error at all.
- **`@disabled` isn't a real Blazor directive.** Boolean HTML attributes like `disabled`/`checked` take a plain attribute name with a C# expression as the value: `disabled="@(condition)"`, no `@` on the attribute itself.
- **Razor directives don't take semicolons.** `@using Foo;` parses into garbage and throws completely unrelated type-conversion errors somewhere else in the file — easily the most confusing failure mode of the whole project.
- **Renaming a folder breaks every namespace.** Blazor derives C# namespaces from folder structure, and `_Imports.razor` is the single file holding all of them. Rename `Components/` to `Views/` and the entire UI tree stops compiling until every `@using` in `_Imports.razor` (and the `App` reference in `Program.cs`) is updated.

None of these are deep bugs. They're all "the framework has a small, specific vocabulary, and stepping outside it fails silently or misleadingly" — which is the real lesson for anyone new to Razor.

---

## Tailwind Without Node

Restyling the UI with Tailwind would normally mean pulling in npm and a PostCSS pipeline — exactly the toolchain Blazor was chosen to avoid. The fix was Tailwind's **standalone CLI**: a single binary, gitignored, wired into the `.csproj` as an MSBuild target that runs `BeforeTargets="Build"`:

```xml
<Target Name="TailwindBuild" BeforeTargets="Build">
  <Exec Command="...\tailwindcss.exe -i input.css -o app.css --minify"
        Condition="Exists('...\tailwindcss.exe')"
        ContinueOnError="WarnAndContinue" />
</Target>
```

`dotnet run` stays the only command anyone needs. The condition means a teammate who hasn't downloaded the 112MB binary yet still gets a working build off the last committed `app.css`. The trade-off is no CSS hot-reload — a new utility class needs a rebuild before it has matching generated CSS.

---

## Small Bugs With Zero Compiler Signal

The two that stuck with me weren't framework quirks, they were typos that compiled clean and ran wrong:

- `NavigationManager.NavigateTo("$/songs/{created.Id}")` — the `$` was inside the string instead of prefixing it, so interpolation never happened. Creating a song silently navigated to the literal path `/songs/{created.Id}` instead of the new record. Valid C#, wrong string, no error anywhere.
- Two tables had a header column with no matching `<td>` — `Description` and `Notes` columns that rendered empty because the cell was never added when the row markup was written. Data that was already saved to the database just never reached the screen.

Both are reminders that a green build proves the code compiles, not that it does what you meant. Worth actually clicking through the create/edit flows, not just trusting `dotnet build`.

---

## What's Next: RAG Over a Music Theory Knowledge Base

The roadmap's last phase is retrieval-augmented generation — chunking music theory references, embedding them with `sentence-transformers`, storing them in ChromaDB, and retrieving the top matches at query time to ground the model's answers instead of relying purely on what it memorized during training. Because the Python service owns the AI layer end to end, this slots in entirely inside `main.py` — the .NET side doesn't change at all.

That's the real argument for the microservice split: each layer can get smarter independently, as long as the boundary between them stays a well-defined HTTP contract.
