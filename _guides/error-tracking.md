---
title: Error Tracking
nav_order: 2
description: How Daylight captures, groups, and manages application errors with fingerprinting and deduplication.
---

Daylight captures exceptions from multiple sources, groups them by fingerprint, and tracks occurrence history, affected users, and resolution status.

## How errors are captured

Daylight hooks into three capture mechanisms automatically:

### Rack middleware

The `Daylight::Middleware::Catcher` sits in your middleware stack and catches any uncaught exception that bubbles up from your application. This is the primary capture mechanism for request-level errors.

### Rails error reporter

On Rails 7.1+, Daylight subscribes to `Rails.error` to capture exceptions reported through the standard Rails error reporting API:

```ruby
Rails.error.handle do
  # exceptions here are captured by Daylight
  risky_operation
end
```

### Manual tracking

You can track errors explicitly anywhere in your code:

```ruby
# Track an error without re-raising
Daylight.track(error, context: { user_id: current_user.id })

# Track and re-raise
Daylight.track!(error, context: { order_id: order.id })
```

## Fingerprinting and deduplication

Every captured error is assigned a fingerprint — a SHA256 hash of:

- Error class name
- First line of the backtrace
- Normalized error message (with dynamic values stripped)

When a new error matches an existing fingerprint, Daylight increments the occurrence count and updates `last_seen_at` rather than creating a duplicate entry. If the error was previously resolved, it is automatically reopened.

## Context extraction

For each occurrence, Daylight captures rich context:

| Context | Source |
|---------|--------|
| Request URL, method, params | Rack request |
| Controller and action | Rails routing |
| User ID | `Current.user` or `context_builder` |
| Tenant | `ApplicationRecord.current_tenant` (if defined) |
| Custom data | Your `context_builder` proc |
| Trace ID | Distributed trace context |

## Backtrace cleaning

Backtraces are filtered to prioritize your application code. Framework and gem lines are removed unless no application lines are present. The first 10 lines are stored as a summary for quick scanning in the error list.

## Error statuses

Each error has a status that you can manage from the dashboard:

| Status | Meaning |
|--------|---------|
| **Open** | Active, unresolved error |
| **Resolved** | Marked as fixed — will reopen if it recurs |
| **Ignored** | Suppressed from the default view |

Bulk operations (resolve, ignore, delete, reopen) are available for managing errors at scale.

## Ignored exceptions

Some exceptions are expected in normal operation. Daylight ignores these by default:

- `ActiveRecord::RecordNotFound`
- `ActionController::RoutingError`
- `ActionController::UnknownFormat`
- `ActionController::InvalidAuthenticityToken`

Add your own:

```ruby
Daylight.configure do |config|
  config.ignored_exceptions += %w[
    Stripe::CardError
    CustomApp::ExpectedError
  ]
end
```

## User impact tracking

Each error occurrence records the `user_id` when available. The error summary shows the total number of distinct affected users, helping you prioritize errors that impact the most people.

## Export

Errors can be exported as CSV or JSON from the dashboard for further analysis or import into other systems.

## Notifications

When a new error is first seen or a resolved error re-opens, Daylight can notify you via email or Slack. See [Notifications](/notifications/) for setup details.
