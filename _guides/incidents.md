---
title: Incidents & Anomaly Detection
nav_order: 7
description: Real-time anomaly detection with AI-investigated incidents, deploy correlation, and root cause analysis.
---

Daylight continuously monitors your application metrics and creates incidents when anomalies are detected. Each incident is automatically investigated by AI to provide root cause analysis and remediation steps.

## Anomaly detection

The anomaly detector runs every 60 seconds (configurable) and checks for four types of anomalies:

### Error spikes

Compares the error count in the last 5 minutes against the hourly baseline. An incident is created when the recent count exceeds the baseline by a configurable multiplier (default: 3x).

| Severity | Threshold |
|----------|-----------|
| **Critical** | > 5x baseline |
| **Warning** | > 3x baseline |

A minimum floor of 5 errors prevents false positives from low-traffic periods.

### New errors

Any error seen for the first time (occurrence count of 1) within the last 5 minutes triggers an info-level incident linked to the specific error.

### Latency spikes

Compares average request duration in the last 5 minutes against the hourly average. Uses the same multiplier thresholds as error spikes.

### Job failure spikes

Compares failed background jobs in the last 5 minutes against the hourly baseline. A minimum floor of 3 failures prevents noise.

## Incident deduplication

Daylight prevents duplicate incidents of the same type within a 30-minute window. If an error spike was already flagged 10 minutes ago, a new incident won't be created until the cooldown expires.

## AI investigation

When an incident is created, Daylight automatically queues an investigation job. The AI gathers context from a window spanning 10 minutes before to 5 minutes after the incident:

- Recent errors and their occurrences
- Primary error details (class, message, backtrace)
- Recent deployments and their git diffs (if GitHub is configured)
- 5xx server errors from the window
- Number of affected users
- 24-hour baseline error rates
- Application context notes from Settings

The investigation report includes:

1. **Summary** — what happened in plain language
2. **Root cause** — best assessment with specific evidence
3. **Deploy correlation** — whether a recent deploy triggered the issue
4. **Blast radius** — users and endpoints affected
5. **Suggested fix** — actionable resolution steps
6. **Prevention** — how to avoid recurrence

## Deploy correlation

If a deployment was recorded within 30 minutes of the incident, Daylight links them and includes the deploy in the investigation. When GitHub is configured, the git diff between the deploy commit and the previous deploy is included in the AI analysis context.

See [Deploys](/deploys/) for how to record deployments.

## Incident statuses

| Status | Meaning |
|--------|---------|
| **Investigating** | Initial state, AI analysis in progress |
| **Open** | Investigation complete, needs human attention |
| **Resolved** | Issue addressed |
| **False alarm** | Not a real problem |

## Configuration

```ruby
Daylight.configure do |config|
  config.anomaly_detection_enabled = true
  config.anomaly_error_spike_threshold = 3.0
  config.anomaly_latency_spike_threshold = 3.0
  config.anomaly_check_interval = 60  # seconds
end
```
