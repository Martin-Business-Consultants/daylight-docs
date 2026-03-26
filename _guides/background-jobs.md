---
title: Background Jobs
nav_order: 5
description: Monitor ActiveJob execution, failures, and duration. Track SolidQueue scheduled tasks and mail delivery.
---

Daylight monitors your background processing infrastructure including ActiveJob, SolidQueue scheduled tasks, and ActionMailer delivery.

## ActiveJob tracking

Daylight subscribes to ActiveSupport instrumentation events for ActiveJob:

- **Enqueue** — when a job is added to the queue
- **Perform** — when a job finishes executing (success or failure)

For each job, Daylight records:

| Field | Description |
|-------|-------------|
| Job class | The ActiveJob class name |
| Queue | Which queue the job ran on |
| Status | `queued`, `completed`, or `failed` |
| Duration | Execution time in milliseconds |
| Error class | Exception class (if failed) |
| Error message | Exception message (if failed) |
| Trace ID | For correlating with other events |

### Failed job error tracking

When a job fails, the exception is automatically recorded as an error through Daylight's tracker. This means job failures appear in your errors dashboard alongside request errors, with the same fingerprinting and deduplication.

## Scheduled task tracking

If you use SolidQueue for recurring tasks, Daylight captures execution events for each scheduled task run:

- Task class and command
- Execution frequency
- Status (completed or failed)
- Duration and any error details

## Mail event tracking

Daylight monitors ActionMailer events:

| Event | Details captured |
|-------|-----------------|
| `deliver` | Mailer class, action, recipients, subject, delivery status, duration |
| `process` | Mailer class, action, processing time |

Failed deliveries are recorded with their error messages for troubleshooting.

## Sampling

In high-throughput applications, you can reduce overhead by sampling job events:

```ruby
Daylight.configure do |config|
  config.sample_rates = {
    jobs: 0.1  # Capture 10% of job events
  }
end
```

Exceptions are always captured regardless of the sampling rate when `always_capture_exceptions` is enabled (the default).
