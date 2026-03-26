---
title: Public API
nav_order: 2
description: Ruby API methods for interacting with Daylight programmatically.
---

Daylight exposes a small public API for manual error tracking, deployment recording, and configuration.

## Error tracking

### `Daylight.track(error, context: {})`

Capture an exception without re-raising it. Use this in rescue blocks where you handle the error but still want to record it.

```ruby
begin
  external_api.call
rescue ExternalApi::Timeout => e
  Daylight.track(e, context: { endpoint: "users", timeout: 30 })
  fallback_response
end
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `error` | Exception | The exception to record |
| `context` | Hash | Optional metadata merged into the occurrence context |

**Returns:** The `Daylight::ErrorRecord` (or `nil` if the error is ignored or sampled out).

### `Daylight.track!(error, context: {})`

Same as `track`, but re-raises the exception after recording it. Useful when you want to record the error at the point it occurs but let it propagate up the call stack.

```ruby
def process_payment(order)
  gateway.charge(order.total)
rescue Gateway::Error => e
  Daylight.track!(e, context: { order_id: order.id })
  # exception is re-raised here
end
```

## Deployment tracking

### `Daylight.deploy!(version:, description:, git_sha:, deployed_by:)`

Record a deployment. All parameters are optional.

```ruby
Daylight.deploy!(
  version: "2.1.0",
  description: "Add user search feature",
  git_sha: "a1b2c3d4",
  deployed_by: "deploy-bot"
)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `version` | String | Version identifier |
| `description` | String | Human-readable deployment description |
| `git_sha` | String | Git commit SHA |
| `deployed_by` | String | Who or what triggered the deploy |

**Returns:** The `Daylight::DeployRecord`.

## Configuration

### `Daylight.configure { |config| ... }`

Configure Daylight at boot time. See [Configuration Reference](/configuration/) for all options.

```ruby
Daylight.configure do |config|
  config.username = "admin"
  config.password = "secret"
  config.sample_rate = 0.5
end
```

### `Daylight.configuration`

Returns the current `Daylight::Configuration` instance for reading config values.

```ruby
Daylight.configuration.sample_rate  # => 0.5
Daylight.configuration.database_path  # => "storage/daylight.sqlite3"
```
