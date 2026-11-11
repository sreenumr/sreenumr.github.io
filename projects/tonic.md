---
title: Tonic
layout: page
---

**Real-time key, scale, and chord detection, written in Go and running on the web, as a PWA, and natively on Android**
Point Tonic at live audio from your microphone or drop in a WAV/MP3, and it tells you the song's key, the diatonic chords for that key at four difficulty tiers (Easy I–IV–V through Full), a ranked list of similar scales, and, independently, the chord playing right now. Key detection uses Krumhansl-Schmuckler key-profile correlation over FFT-derived chroma vectors on a rolling ~6s window with hysteresis, so the displayed key stays stable instead of flickering with each chord change; a separate ~1.5s window tracks the current chord. All DSP and music theory lives in one shared Go engine, served to a plain HTML/CSS/JS frontend with no build step. The same code ships as a desktop web app, an installable PWA (served over self-signed HTTPS for LAN installs), and a fully offline Android app that embeds the Go server via `gomobile` inside a thin Kotlin WebView shell with native `AudioRecord` capture. Nothing is recorded or uploaded anywhere.
**GitHub:** https://github.com/sreenumr/Tonic
