---
title: Getting Started
nav_order: 1
description: Install and configure Daylight in your Rails application in under five minutes.
---

Daylight is a Rails engine that gives you error tracking, performance monitoring, and AI-powered insights out of the box. It stores everything in a local SQLite database — no external services required.

## Requirements

- Ruby >= 3.1
- Rails >= 7.0

## Installation

Add Daylight to your Gemfile:

```ruby
gem "daylight"
```
{: data-title="Gemfile"}

Run bundler:

```bash
bundle install
```

## Mount the engine

Add the Daylight routes to your application:

```ruby
Rails.application.routes.draw do
  mount Daylight::Engine => "/daylight"

  # ... your other routes
end
```
{: data-title="config/routes.rb"}

That's it. Start your Rails server and visit `/daylight` to see the dashboard.

## Configuration

Create an initializer to customize Daylight's behavior:

```ruby
Daylight.configure do |config|
  # Protect the dashboard with HTTP Basic Auth
  config.username = "admin"
  config.password = "secret"

  # Set the minimum log level to capture (default: WARN)
  config.log_capture_level = 2

  # Sampling rate for high-traffic apps (0.0 to 1.0)
  config.sample_rate = 1.0

  # Always capture exceptions, even when sampled out
  config.always_capture_exceptions = true

  # Add custom context to every captured event
  config.context_builder = ->(request) {
    { user_id: request.env["warden"]&.user&.id }
  }
end
```
{: data-title="config/initializers/daylight.rb"}

See the [Configuration Reference](/configuration/) for all available options.

## What happens automatically

Once mounted, Daylight immediately begins capturing:

- **Uncaught exceptions** via Rack middleware and the Rails error reporter
- **HTTP requests** with timing, status codes, and controller actions
- **Database queries** that exceed the slow query threshold (50ms default)
- **Background jobs** via ActiveSupport instrumentation
- **Cache operations** including hit/miss tracking
- **External HTTP calls** made through Net::HTTP
- **Application logs** at your configured level
- **Mail delivery** events from ActionMailer

All events are linked together with distributed trace IDs, so you can follow a request from the initial HTTP call through every database query, cache lookup, and background job it triggers.

## Database storage

Daylight stores all data in a SQLite database at `storage/daylight.sqlite3` within your Rails app. The database schema is automatically created on first startup — no migrations to run.

Data is retained for 30 days by default. You can adjust this in the Settings page of the dashboard or trigger a manual cleanup.

## Securing the dashboard

For production, always set authentication credentials. Daylight supports two approaches:

**Option 1: Initializer config** (shown above)

**Option 2: Rails encrypted credentials**

```bash
rails credentials:edit
```

```yaml
daylight:
  username: admin
  password: your-secure-password
```
{: data-title="config/credentials.yml.enc"}

Daylight checks credentials in this order: explicit config, then Rails credentials, then no auth (open access).

{% include alert.html type="warning" content="Never deploy Daylight without authentication in production. The dashboard exposes error details, request data, and application internals." %}

## Next steps

- [Error Tracking](/error-tracking/) — understand how errors are captured and grouped
- [Request Monitoring](/request-monitoring/) — explore performance metrics and request waterfalls
- [AI-Powered Insights](/ai-insights/) — set up AI scanning for performance and security issues
- [Configuration Reference](/configuration/) — full list of every configuration option
