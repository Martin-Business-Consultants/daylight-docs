---
title: Distributed Tracing
nav_order: 10
description: Understand how Daylight links requests, queries, jobs, and other events with trace IDs.
---

Daylight assigns a trace ID to every request and background job, linking all events that occur within that execution context. This gives you a complete picture of what happened during any single operation.

## How it works

When a Rails request begins, Daylight generates a unique trace ID and stores it in `Thread.current[:daylight_trace_id]`. Every subscriber (queries, HTTP calls, cache operations, logs, exceptions) reads this trace ID and attaches it to its recorded event.

Background jobs get a fresh trace ID when they start executing, creating a separate trace for each job run.

The trace ID is cleared when the request or job completes.

## What gets linked

Within a single trace, you can see:

- The HTTP request with timing breakdown
- Every database query with SQL and duration
- External HTTP calls to third-party APIs
- Cache reads (hits and misses), writes, and deletes
- Application log entries
- Any exceptions raised
- Mail delivery events triggered by the request

## Using the waterfall view

From the Requests dashboard:

1. Click on a route pattern to see individual requests
2. Click on a specific request to see its full waterfall
3. Events are displayed chronologically with their duration

This makes it straightforward to answer questions like:

- "Why was this request slow?" — check for slow queries or external API calls
- "What queries did this endpoint run?" — see the full query list with source locations
- "Did this request hit the cache?" — check cache operations and hit/miss ratio

## Sampling and trace completeness

When sampling is enabled, the sampling decision is made once at the start of each request. If a request is sampled in, all its events are captured. If sampled out, detailed events (queries, cache, logs) are skipped — but exceptions are still captured when `always_capture_exceptions` is enabled.

This means every captured trace is complete — you never see a partial request with missing queries.
