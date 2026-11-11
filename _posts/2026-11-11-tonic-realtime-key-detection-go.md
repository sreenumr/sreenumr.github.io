---
title: "Tonic: One Go Codebase for Real-Time Key Detection on Web, PWA, and Android"
date: 2026-11-11
categories: [Engineering, Go, Android, Audio]
---

Tonic listens to music, either live through the microphone or from an uploaded WAV/MP3, and tells you what key it's in, which chords belong to that key, and which chord is playing right now. The brief was "Shazam, but for scales." The part I found most interesting turned out to be less the detection itself and more getting **one Go codebase** to ship unchanged as a web app, an installable PWA, and a fully offline native Android app.

---

## How Key Detection Works

There's no machine learning here. It's classic signal processing plus a well-known piece of music cognition research:

1. **Chroma extraction:** a Hann-windowed FFT folds the spectrum into a 12-bin *chroma vector*, which measures how much energy sits on each pitch class (C, C♯, D, …) regardless of octave.
2. **Krumhansl-Schmuckler correlation:** each of the 24 candidate keys (12 tonics × major/minor) has a *key profile*, an empirically derived weighting of how strongly each pitch class "belongs" in that key. Correlate the chroma vector against all 24, and the highest score wins. The rest of the ranking becomes the "similar scales" list.
3. **Diatonic chords:** once the key is known, the triads follow directly from music theory. They're shown at four tiers: Easy (I IV V), Medium (+vi), Hard (+ii), and Full (all seven).

I verified the pipeline the boring way first: pure sine tones checked that the right pitch class lit up, and a synthetic C-major WAV checked the whole thing end to end. It came back "C Major" at ~94% confidence, with the correct chords.

---

## Key vs. Chord: Two Windows, Not One

The first live version recomputed every ~500ms over a 3-second window and labelled the result "scale." In practice the label changed every time the *chord* changed, so it looked like the song kept switching keys.

The fix was to split it into two independent estimates with different time horizons:

- **Key/scale:** correlated over a rolling **~6s** window, with **hysteresis**: a new key has to beat the currently displayed one by a margin before it takes over. It reads as a stable, slow-moving "this song is in X."
- **Current chord:** direct chroma-vs-triad-template matching (36 candidates: 12 roots × major/minor/diminished) over just the trailing **~1.5s**, independent of key context. It gets its own clearly labelled "chord playing now" badge.

I tested this with a synthetic C–F–G–Am progression. The chord badge tracked each change in real time while the key held steady at C Major. One honest caveat: if a single vi chord is sustained alone for the entire 6s window, the key drifts to the relative minor. That's arguably the right answer for that input, and real recordings, with basslines and other voices reinforcing the tonic, are far more stable.

---

## One Engine, Three Targets

The whole layout comes down to one rule: **everything that matters is platform-agnostic, and the platform-specific code stays thin.**

```
internal/theory    key profiles, correlation, diatonic chords
internal/dsp       FFT -> chroma
internal/audio     WAV/MP3 decode
internal/analysis  file + streaming analyzers
internal/api       NewMux(fs.FS) - HTTP + WebSocket
assets/web         THE frontend (HTML/CSS/JS, no build step)
   -- shared above, platform-specific below --
cmd/server         desktop: HTTP :8080 + self-signed HTTPS :8443
mobile/            gomobile entry point: Start(port) on 127.0.0.1
android/           Kotlin WebView shell
```

The refactor that made this work was small. I moved static files from `http.Dir("web")` to `//go:embed`, and changed `api.NewMux` to take an `fs.FS`. A phone has no `web/` directory on disk, but an embedded filesystem works anywhere. After that, `gomobile bind` produces an AAR, and the Android app just calls `Mobile.start(8080)` on a background thread and points a WebView at `http://127.0.0.1:8080/`. The entire detection engine runs inside the app's own process, with no PC and no network.

The Go side never needs cgo or native audio libraries. Capture happens in the browser (Web Audio API + `AudioWorklet`), which streams float32 PCM over a WebSocket, and Go does 100% of the analysis.

---

## The PWA Needed Real HTTPS

Service workers and proper "Install app" prompts require a secure origin, and `localhost` is the *only* plain-HTTP exception. A phone visiting the PC's LAN IP over HTTP only gets offered a bookmark, not an installable app.

So the desktop server also generates (and reuses) a long-lived self-signed certificate and serves HTTPS on `:8443`, printing the LAN URLs at startup. The one-time cost is clicking through an untrusted-certificate warning on the phone. After that, it behaves like a normal installed app with an offline-cached shell. Because the manifest and service worker live under `assets/web/`, the Android app's embedded server serves them too, at no extra cost.

---

## The Android Mic Bug That Wasn't Mine

On the native app, live listening failed with `NotReadableError: Could not start audio source`, even though `RECORD_AUDIO` was granted *and* the WebView's `onPermissionRequest` confirmed the grant.

The first step was making the failure diagnosable. The original JS wrapped `getUserMedia`, `AudioContext`, the worklet, *and* the WebSocket in a single try/catch, so every failure got reported as "microphone access failed." I split those up and forwarded WebView console output to `adb logcat`, which surfaced the real cause:

```
AttributionAndPermissionUtils: ... Permission hard denied for client
attribution [uid ..., packageName "<unknown>"]
```

This is a known WebView/Chromium issue on API 31+: WebView's internal audio capture doesn't always identify itself correctly to Android's permission-attribution system, so the OS hard-denies the mic at the hardware level. Updating WebView didn't help.

The fix was to stop using WebView for capture at all. Kotlin now records mono 16-bit PCM with native `AudioRecord`, converts it to the *same* little-endian float32 wire format the browser path uses, and streams it to the *same* `/ws` endpoint over OkHttp. The page exposes a small JS bridge (`window.AndroidAudio`), and `app.js` picks the native path when that bridge exists and `getUserMedia` everywhere else. The rendering code is identical either way. The Go engine didn't change at all.

---

## Measure Before Theorizing

The last thread is still open. Live detection in the native app felt less stable than in the browser. My first guess was that the OEM's voice-tuned mic processing (AGC, noise suppression) was mangling harmonic content, so I switched to `AudioSource.UNPROCESSED`.

Rather than eyeball it, I added level telemetry to the capture loop (peak, RMS and clipped samples, logged once per second). The numbers disproved the theory straight away. On my device, `UNPROCESSED` delivered near-silence (a peak of ~20–300 out of 32768, roughly −40 dB) even while music was playing. The signal was drowning in the noise floor, which explains the instability far better than distortion would. I reverted to `MIC`, and the next step is to retest with fresh telemetry, then check the capture sample rate and chunk framing if the levels look healthy.

The lesson from this one is that the shared Go engine was untouched, so any difference between native and web *had* to come from the audio reaching it. Adding a few lines of telemetry and looking at real numbers settled the question faster than more guessing would have.

---

## What's Next

- Finish the native-audio accuracy investigation described above.
- More modes beyond major/natural minor (Dorian, Mixolydian, …). The profile table is already structured so these are just additional entries.
- A native desktop shell (e.g. Wails) around the same backend and frontend.
