---
title: Routes & Endpoints
nav_order: 3
description: Complete list of all dashboard routes and API endpoints provided by the Daylight engine.
---

All routes are mounted under your chosen path (typically `/daylight`). The examples below assume the default mount point.

## Errors

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight` | Dashboard home (redirects to errors) |
| `GET` | `/daylight/errors` | Error list with filters and search |
| `GET` | `/daylight/errors/:id` | Error detail with occurrences |
| `PATCH` | `/daylight/errors/:id` | Update error status |
| `DELETE` | `/daylight/errors/:id` | Delete error and all occurrences |
| `POST` | `/daylight/errors/batch` | Bulk operations (resolve, ignore, delete, reopen) |
| `GET` | `/daylight/errors/export` | Export errors as CSV or JSON |

## Requests

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/requests` | Request metrics aggregated by route pattern |
| `GET` | `/daylight/requests/export` | Export request data |

## Database queries

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/queries` | Slow query list and N+1 patterns |

## Background jobs

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/jobs` | Job execution history |

## Scheduled tasks

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/scheduled_tasks` | SolidQueue recurring task history |

## Mail events

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/mail_events` | ActionMailer delivery and processing events |

## Logs

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/logs` | Captured application logs |

## HTTP requests

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/http_requests` | External HTTP call tracking |

## Cache

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/cache` | Cache operation metrics |

## Deploys

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/deploys` | Deployment history |
| `POST` | `/daylight/deploys` | Record a deployment |

## Incidents

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/incidents` | Incident list with status filters |
| `GET` | `/daylight/incidents/:id` | Incident detail with AI investigation |
| `PATCH` | `/daylight/incidents/:id` | Update incident status |

## Solutions

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/solutions` | AI-generated solution list |
| `GET` | `/daylight/solutions/:id` | Solution detail with chat history |
| `PATCH` | `/daylight/solutions/:id` | Update solution status |
| `POST` | `/daylight/solutions/:id/chat` | Send message to refine solution |
| `POST` | `/daylight/solutions/:id/push` | Push solution as GitHub PR |
| `POST` | `/daylight/solutions/:id/regenerate` | Regenerate solution from scratch |
| `POST` | `/daylight/solutions/generate` | Generate solutions for all open issues |

## Settings

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/settings` | Configuration page |
| `PATCH` | `/daylight/settings` | Update settings |
| `POST` | `/daylight/settings/cleanup` | Run data cleanup |
| `POST` | `/daylight/settings/test_notification` | Send test email/Slack |
| `POST` | `/daylight/settings/run_performance_scan` | Trigger performance scan |
| `POST` | `/daylight/settings/run_security_scan` | Trigger security scan |
| `POST` | `/daylight/settings/toggle_bullet_diagnostic` | Start N+1 diagnostic window |
| `POST` | `/daylight/settings/stop_bullet_diagnostic` | Stop diagnostic window |
| `PATCH` | `/daylight/settings/performance_issues/:id` | Dismiss performance issue |
| `PATCH` | `/daylight/settings/security_issues/:id` | Dismiss security issue |

## Health

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/daylight/health` | Health check endpoint |
