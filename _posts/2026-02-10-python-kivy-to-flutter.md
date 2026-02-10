---
title: "Why I Switched from Python Kivy to Flutter — and Finished the App"
date: 2026-02-10
categories: [Mobile, Engineering, Lessons]
---

This project started with a simple goal:  
convert files into QR codes and reconstruct them back reliably on a mobile device.

On paper, it sounded straightforward.

In practice, the **technology choice mattered more than the logic**.

---

## Initial Approach: Python + Kivy

I initially chose **Python with Kivy**.

The reasoning was familiar:
- I already knew Python well
- The core logic (binary conversion, encoding/decoding) was easier to prototype
- One language across backend and mobile felt efficient

Reality disagreed.

### The Actual Problems

- Android builds relied on **WSL and cross-compilation**
- Build times ran into **hours**
- Tooling failures were opaque and poorly documented
- Most of my time went into *fighting the build system*, not building features

After **weeks spent just trying to get a stable build**, progress was effectively zero.

The app logic wasn’t the bottleneck.  
The ecosystem was.

---

## The Switch: Flutter

At that point, I made a hard decision: **drop Kivy and switch to Flutter**.

This wasn’t about language preference.  
It was about **delivery**.

### What Changed Immediately

- Predictable build tooling
- Clear Android/iOS support
- Fast iteration cycles
- Excellent QR and binary-handling libraries
- Strong documentation and community examples

Within **one week**, the app was:
- Functional
- Buildable
- Testable on real devices

---

## Final Outcome

- Built a **Flutter-based mobile application** that converts files into QR codes and reconstructs them back
- Implemented **binary data conversion → string encoding → QR generation**
- Achieved **successfull file conversion**
- Ensured accurate file recovery with validation and error handling
- Delivered a usable product instead of a half-working prototype

---

## The Real Lesson

This project reinforced a hard truth:

Python was great for experimentation.  
Flutter was better for execution.

Switching tools felt like a setback at first.  
In reality, it was the reason the project shipped at all.

---

## What I’d Do Again

- Prototype fast
- Kill bad tooling early
- Optimise for **delivery, not familiarity**
- Treat build systems as first-class requirements

Sometimes the smartest optimisation isn’t in code —  
it’s in **walking away from the wrong stack**.
