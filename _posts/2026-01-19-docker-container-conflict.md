---
layout: post
title: "The Docker Container Name Conflict That Broke Redeployments"
categories: [devops]
tags: [docker, automation]
excerpt: "Why idempotency matters more than your first deploy."
---

## Problem
Redeployments failed with:
`container name already in use`.

## Why This Happened
The deploy script assumed a clean host.
Production never is.

## Fix
Make cleanup explicit and safe:
- Stop container if exists
- Remove container if exists
- Then deploy

## Takeaway
If your deployment isn’t idempotent, it’s fragile.
