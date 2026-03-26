---
title: Configuration Reference
nav_order: 1
description: Complete reference for all Daylight configuration options, both initializer-based and dashboard-based.
---

Daylight is configured through two mechanisms: a Ruby initializer for boot-time settings, and the dashboard Settings page for runtime settings stored in the database.

## Initializer options

Set these in `config/initializers/daylight.rb`:

```ruby
Daylight.configure do |config|
  # ...
end
```

### Authentication

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `username` | String | `nil` | HTTP Basic Auth username for the dashboard |
| `password` | String | `nil` | HTTP Basic Auth password for the dashboard |

If not set, Daylight checks `Rails.application.credentials.dig(:daylight, :username)` and `:password`. If neither is configured, the dashboard has no authentication.

### Database

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `database_path` | String | `storage/daylight.sqlite3` | Path to the SQLite database file |

### Capture behavior

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `auto_capture` | Boolean | `true` | Automatically install middleware and subscribers |
| `ignored_exceptions` | Array | See below | Exception classes to skip |
| `log_capture_level` | Integer | `2` (WARN) | Minimum log level to record (0=DEBUG, 1=INFO, 2=WARN, 3=ERROR, 4=FATAL) |
| `context_builder` | Proc | `nil` | Lambda receiving the Rack request, returns a hash merged into occurrence context |
| `always_capture_exceptions` | Boolean | `true` | Capture exceptions even when the request is sampled out |

### Default ignored exceptions

```ruby
%w[
  ActiveRecord::RecordNotFound
  ActionController::RoutingError
  ActionController::UnknownFormat
  ActionController::InvalidAuthenticityToken
]
```

### Sampling

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `sample_rate` | Float | `1.0` | Global sampling rate (0.0 to 1.0) |
| `sample_rates` | Hash | `{}` | Per-event-type rates, e.g., `{ requests: 1.0, jobs: 0.1 }` |

### Anomaly detection

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `anomaly_detection_enabled` | Boolean | `true` | Enable/disable anomaly detection |
| `anomaly_error_spike_threshold` | Float | `3.0` | Multiplier over baseline to trigger error spike |
| `anomaly_latency_spike_threshold` | Float | `3.0` | Multiplier over baseline to trigger latency spike |
| `anomaly_check_interval` | Integer | `60` | Seconds between anomaly checks |

## Dashboard settings

These are configured in the Settings page and stored in the `daylight_settings` database table.

### AI configuration

| Setting | Description |
|---------|-------------|
| `gemini_api_key` | Google Gemini API key |
| `anthropic_api_key` | Anthropic Claude API key |
| `default_ai_model` | Model ID for AI features |
| `ai_context_notes` | Free-text notes about your app provided to AI as context |

### GitHub integration

| Setting | Description |
|---------|-------------|
| `github_repo_url` | Repository URL (e.g., `https://github.com/org/repo`) |
| `github_default_branch` | Target branch for PRs (e.g., `main`) |
| `github_api_token` | Personal access token with `repo` scope |

### Notifications

| Setting | Description |
|---------|-------------|
| `notification_emails` | Comma-separated email addresses |
| `slack_webhook_url` | Slack incoming webhook URL |

### Performance thresholds

| Setting | Default | Description |
|---------|---------|-------------|
| `slow_request_threshold_ms` | — | Requests slower than this trigger performance errors |
| `slow_query_threshold_ms` | — | Query recording threshold |

### Scanning schedules

| Setting | Description |
|---------|-------------|
| `performance_scan_enabled` | Enable scheduled performance scans |
| `performance_scan_interval` | Scan frequency (e.g., `6h`, `12h`, `daily`, `weekly`) |
| `security_scan_enabled` | Enable scheduled Brakeman scans |
| `security_scan_interval` | Scan frequency |
| `security_scan_min_confidence` | Minimum Brakeman confidence to record (`high`, `medium`, `weak`) |
| `solutions_scan_enabled` | Automatically generate AI solutions for new issues |

### Data retention

| Setting | Default | Description |
|---------|---------|-------------|
| `retention_days` | `30` | Days to keep time-series data before cleanup |
| `sample_rate` | `1.0` | Runtime override for global sampling |
