---
title: MusicMentor
layout: page
---

**AI-powered music learning app built with ASP.NET Core, Blazor, and a Python AI microservice**
Song key detection, scale and chord retrieval, chord progression generation, and transposition, all backed by a deterministic C# music theory engine. A separate Python FastAPI service handles natural-language music theory Q&A through a local LLM (Ollama), reachable from .NET behind a swappable `IAiMusicService` interface so the model backing it can change with a one-line config edit. SQLite + EF Core persist songs and setlists; the UI is a Blazor Web App styled with Tailwind CSS via its standalone CLI (no Node).
**GitHub:** https://github.com/sreenumr/MusicMentor
