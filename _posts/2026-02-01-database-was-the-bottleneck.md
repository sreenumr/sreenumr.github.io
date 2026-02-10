---
title: "The Database Was the Bottleneck, Not the API"
date: 2026-02-01
categories: [Performance, Databases]
---

**Problem**

API performance degraded under load.
Scaling the service didn’t help.

Requests piled up.
Response times spiked.

**Diagnosis**

- Missing indexes on high-frequency queries
- N+1 query patterns
- Connection pool exhaustion

**Solution**

- Added targeted indexes
- Reduced query count per request
- Batch queries

**Result**

Lower response times without scaling infrastructure.

**Lesson**

Scaling compute won’t fix inefficient queries.
