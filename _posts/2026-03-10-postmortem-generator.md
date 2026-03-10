---
title: "Python based AI incident document builder"
date: 2026-03-10
categories: [Engineering, Python]
---

# Building a Local AI Incident Postmortem Generator

I recently built a small project to explore how LLMs can help automate incident documentation. The idea was simple: take structured incident events, reconstruct the timeline, and generate a clean postmortem report automatically.

The system is a local-first CLI tool that combines **DuckDB, Git repository context, a lightweight local LLM (via Ollama), and Redis caching**.

---

## What the System Does

The workflow looks like this:

Incident Events → DuckDB Timeline
↓
Git Repository Context
↓
Context Builder
↓
Local LLM (Ollama)
↓
Structured Postmortem Report


1. Incident events (alerts, deploys, chat logs) are ingested and stored in **DuckDB**.
2. A chronological **timeline** is reconstructed using SQL.
3. The system extracts **Git commits and changed files** within the incident window.
4. The timeline and repo context are passed to a **local LLM** to generate a structured postmortem report.
5. The result is saved as Markdown and stored in the database.

---

## Key Design Idea

Instead of letting the LLM "figure things out", the system separates responsibilities:

- **DuckDB** → computes the timeline and facts
- **Git** → provides code change context
- **LLM** → narrates the incident in human-readable form

This keeps the system deterministic while still benefiting from AI-generated summaries.

---

## Challenges Encountered

A few things didn't work well initially:

- **Small local models produced messy JSON output**  
  → solved by adding schema validation and sanitizing responses.

- **Dumping raw Git diffs confused the model**  
  → solved by extracting only commit messages and changed files.

- **Caching reports by incident ID caused stale results**  
  → solved by hashing the timeline and repo context to generate a content fingerprint.

---

## Redis Caching

To avoid unnecessary LLM calls, the project uses **content-based caching**:

hash(timeline_text + repo_context)

If the timeline or repo context changes, the cache invalidates automatically.

---

## Takeaways

This project reinforced a key lesson when building AI tools:

> LLMs should generate narrative facts from structured inputs, not replace structured systems that compute those inputs.
