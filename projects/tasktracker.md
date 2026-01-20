---
layout: page
title: TaskTrackerAPI
---

## What this is
A production-style FastAPI service deployed via CI/CD to AWS EC2.

## Why I built it
To practice end-to-end DevOps ownership:
build → test → scan → deploy → observe.

## Architecture
- FastAPI
- Docker
- GitLab CI/CD
- Terraform
- AWS EC2, RDS, S3

## What works
- Idempotent deployments
- Security scans in pipeline
- Healthcheck-based validation

## What broke (and how I fixed it)
- Container name conflicts
- Incorrect binding (`127.0.0.1`)
- Masked CI variables

## Links
- GitHub repo
- Blog posts related to this project
