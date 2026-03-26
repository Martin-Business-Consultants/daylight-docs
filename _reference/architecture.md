---
title: Architecture
nav_order: 5
description: How Daylight is structured internally — engine isolation, subscriber system, AI pipeline, and data flow.
---

This page describes Daylight's internal architecture for contributors and advanced users who want to understand how the pieces fit together.

## Engine isolation

Daylight is a Rails engine mounted as an isolated namespace. This means:

- All models, controllers, and routes are scoped under `Daylight::`
- The engine uses its own SQLite database, completely separate from your application database
- Static assets are served from the engine's pre-built bundle at `/daylight/assets/`
- No migrations pollute your application's migration history — the schema auto-creates on first boot

## Data flow

```
Rails App
  │
  ├─ Middleware (Catcher) ──────────┐
  ├─ Rails.error.subscribe ─────────┤
  ├─ ActiveSupport Notifications ───┤
  │   ├─ process_action.action_controller
  │   ├─ sql.active_record           ├──▶ Tracker / Subscribers
  │   ├─ enqueue.active_job          │         │
  │   ├─ perform.active_job          │         ▼
  │   ├─ request.net_http            │    SQLite Database
  │   ├─ cache_*.active_support      │    (storage/daylight.sqlite3)
  │   ├─ *.action_mailer             │         │
  │   └─ perform.solid_queue ────────┘         ▼
  │                                     Dashboard (Svelte + Inertia.js)
  └─ Manual API (Daylight.track)
```

## Subscriber system

Daylight uses ActiveSupport::Notifications to subscribe to Rails instrumentation events. Each subscriber is a dedicated class in `lib/daylight/subscribers/`:

| Subscriber | Events | Records to |
|------------|--------|------------|
| `RequestSubscriber` | `process_action.action_controller` | `daylight_requests` |
| `QuerySubscriber` | `sql.active_record` | `daylight_queries` |
| `JobSubscriber` | `enqueue.active_job`, `perform.active_job` | `daylight_jobs` |
| `HttpSubscriber` | `request.net_http` | `daylight_http_requests` |
| `CacheSubscriber` | `cache_read/write/delete.active_support` | `daylight_cache_events` |
| `LogSubscriber` | Logger interception | `daylight_logs` |
| `MailSubscriber` | `deliver/process.action_mailer` | `daylight_mail_events` |
| `ScheduledTaskSubscriber` | `perform.solid_queue` | `daylight_scheduled_tasks` |

All subscribers check the sampling decision before writing. The request subscriber additionally runs the anomaly detector and checks if scheduled scans are due.

## Trace context

A trace ID is generated at the start of each request and stored in `Thread.current[:daylight_trace_id]`. Every subscriber reads this value and attaches it to its records. Background jobs get a fresh trace ID.

This enables the waterfall view: given a trace ID, Daylight queries all event tables to reconstruct the full execution timeline.

## AI pipeline

```
Performance Scanner ──┐
                      ├──▶ Issues (performance_issues / security_issues)
Security Scanner ─────┘         │
                                ▼
                      Solution Generator
                          │
                          ├──▶ Draft Solution
                          │       │
                          │   User Chat (refine)
                          │       │
                          └──▶ Push to GitHub (PR)
```

The AI pipeline is entirely optional and requires API keys. Each stage runs as a background job:

1. **PerformanceScanJob** — analyzes query data from last 24h
2. **SecurityScanJob** — runs Brakeman static analysis
3. **SolutionGenerationJob** — generates fixes for open issues
4. **InvestigateIncidentJob** — analyzes anomaly-triggered incidents

## Frontend

The dashboard uses Svelte 5 with Inertia.js for server-driven rendering. Controllers pass props directly to Svelte page components — no separate API layer. Assets are pre-built with Vite and shipped with the gem.

## Graceful degradation

Every tracking operation wraps in `begin/rescue` to ensure Daylight never interferes with your application:

- Database connectivity is checked before writes
- Notification delivery failures are silently logged
- AI features work without API keys (scanning is simply skipped)
- Missing optional gems (Bullet, Brakeman) disable their respective features
