---
layout: post
title: "The Docker Container Name Conflict That Broke Redeployments"
categories: [devops]
tags: [docker, automation]
excerpt: "Why idempotency matters more than your first deploy."
---

## Context

While building a FastAPI service with GitLab CI/CD, I ran into several issues
that weren’t obvious from documentation.

This post captures the key failures and fixes.

---

## Issue 1: App not reachable after deployment



**Symptom**
- Container started successfully
- `curl` returned empty response


**Cause**
- App was bound to `127.0.0.1` inside the container

## Fix

```bash
gunicorn main:app -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

