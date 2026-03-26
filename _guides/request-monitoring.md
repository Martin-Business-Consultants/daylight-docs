---
title: Request Monitoring
nav_order: 3
description: Track HTTP request performance with timing breakdowns, Apdex scoring, and full request waterfalls.
---

Daylight records every Rails request with detailed timing breakdowns, then aggregates the data by route pattern to surface performance trends and outliers.

## What gets captured

For each request, Daylight records:

- HTTP method and path
- Controller and action
- Response status code
- Total duration, database time, and view rendering time
- Query count per request
- Request format (HTML, JSON, etc.)
- Client IP address
- Whether an N+1 query was detected
- Distributed trace ID

## Route pattern aggregation

Raw paths like `/users/42` and `/users/789` are normalized to `/users/:id` for aggregation. UUIDs are similarly collapsed to `:uuid`. This gives you per-endpoint metrics without noise from dynamic segments.

The requests dashboard shows aggregate metrics for each route pattern:

- **Average duration** across all requests
- **P95 latency** — the 95th percentile response time
- **Max duration** — worst-case response time
- **Throughput** — requests per minute
- **Apdex score** — user satisfaction index

## Apdex scoring

Daylight calculates an [Apdex score](https://en.wikipedia.org/wiki/Apdex) for each route:

| Response time | Classification |
|---------------|----------------|
| < 500ms | Satisfied |
| 500ms – 2000ms | Tolerating |
| > 2000ms | Frustrated |

The Apdex score ranges from 0 (all frustrated) to 1.0 (all satisfied).

## Request waterfall

Click into any individual request to see its full execution waterfall:

- **Database queries** — every SQL statement with duration and source location
- **External HTTP calls** — outbound API requests with timing
- **Cache operations** — hits, misses, writes, and deletes
- **Application logs** — log entries from this request
- **Exceptions** — any errors raised during the request

All events are linked by trace ID, giving you a complete picture of what happened during a single request.

## N+1 detection

Daylight tracks normalized SQL patterns within each request. If the same query pattern executes 5 or more times in a single request, it is flagged as an N+1 query. The request record is marked with `n_plus_one = true` for easy filtering.

For deeper N+1 analysis with AI-generated fix suggestions, see [AI-Powered Insights](/ai-insights/).

## Slow request tracking

Requests that exceed the slow request threshold (configurable in Settings) are automatically logged as performance-severity errors. This lets you track slow endpoints alongside your application errors with the same resolution workflow.

## Deploy overlay

If you track deployments, the requests dashboard overlays deploy markers on latency and throughput charts. This makes it easy to correlate performance changes with specific releases.

See [Deploys](/deploys/) for how to record deployments.
