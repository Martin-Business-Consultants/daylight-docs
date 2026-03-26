---
title: Database Schema
nav_order: 4
description: Complete schema reference for all Daylight database tables.
---

Daylight uses a dedicated SQLite database, separate from your application database. All tables are prefixed with `daylight_` and auto-created on first startup.

## daylight_errors

The primary error aggregation table. Each row represents a unique error fingerprint.

| Column | Type | Default | Description |
|--------|------|---------|-------------|
| `fingerprint` | string | — | SHA256 hash (unique, indexed) |
| `error_class` | string | — | Exception class name |
| `message` | text | — | Error message |
| `backtrace_summary` | text | — | First 10 lines of backtrace |
| `occurrences_count` | integer | `1` | Number of times this error occurred |
| `status` | string | `open` | `open`, `resolved`, or `ignored` (indexed) |
| `severity` | string | `error` | `error`, `performance`, or `warning` |
| `handled` | boolean | — | Whether the exception was caught |
| `source` | string | — | Capture source: `middleware`, `job`, `rails_error_reporter` |
| `first_seen_at` | datetime | — | When first recorded (indexed) |
| `last_seen_at` | datetime | — | Most recent occurrence (indexed) |
| `affected_users_count` | integer | `0` | Distinct users who experienced this error |
| `avg_duration_ms` | float | — | Average duration (performance issues) |
| `max_duration_ms` | float | — | Maximum duration (performance issues) |
| `threshold_exceeded_count` | integer | — | Times performance threshold was exceeded |

## daylight_occurrences

Individual error instances linked to their parent error.

| Column | Type | Description |
|--------|------|-------------|
| `error_id` | integer | Foreign key to `daylight_errors` (indexed) |
| `backtrace` | text | Full backtrace |
| `context` | JSON | Request context, user info, custom data |
| `request_url` | string | URL where the error occurred |
| `request_method` | string | HTTP method |
| `user_id` | string | Affected user identifier (indexed) |
| `request_id` | integer | Linked request record (indexed) |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | When this occurrence happened (indexed) |

## daylight_requests

HTTP request records with performance metrics.

| Column | Type | Default | Description |
|--------|------|---------|-------------|
| `method` | string | — | HTTP method |
| `path` | string | — | Request path |
| `controller_action` | string | — | `controller#action` (indexed) |
| `status_code` | integer | — | HTTP response code |
| `duration_ms` | float | — | Total request time (indexed) |
| `db_duration_ms` | float | `0` | Time in database |
| `view_duration_ms` | float | `0` | Time rendering views |
| `query_count` | integer | `0` | Number of SQL queries |
| `route_pattern` | string | — | Normalized route (indexed) |
| `n_plus_one` | boolean | — | N+1 query detected |
| `format` | string | — | Response format |
| `ip` | string | — | Client IP address |
| `trace_id` | string | — | Distributed trace ID (indexed) |
| `occurred_at` | datetime | — | Request timestamp (indexed) |

## daylight_queries

Slow database query records.

| Column | Type | Description |
|--------|------|-------------|
| `sql` | text | Full SQL statement |
| `normalized_sql` | string | Parameterized SQL for grouping (indexed) |
| `duration_ms` | float | Execution time (indexed) |
| `source_location` | string | Application code location |
| `controller_action` | string | Controller and action |
| `request_path` | string | Request that triggered the query |
| `request_id` | integer | Linked request (indexed) |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Query timestamp (indexed) |

## daylight_jobs

Background job execution records.

| Column | Type | Description |
|--------|------|-------------|
| `job_class` | string | ActiveJob class name (indexed) |
| `queue` | string | Queue name |
| `status` | string | `queued`, `completed`, or `failed` |
| `duration_ms` | float | Execution time |
| `error_message` | text | Exception message (if failed) |
| `error_class` | string | Exception class (if failed) |
| `enqueued_at` | datetime | When the job was queued |
| `completed_at` | datetime | When the job finished |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Event timestamp (indexed) |

## daylight_http_requests

External HTTP call records.

| Column | Type | Description |
|--------|------|-------------|
| `method` | string | HTTP method |
| `url` | string | Full URL |
| `host` | string | Target hostname (indexed) |
| `status_code` | integer | Response code |
| `duration_ms` | float | Request time (indexed) |
| `controller_action` | string | Triggering controller action |
| `request_path` | string | Parent request path |
| `request_id` | integer | Linked request (indexed) |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Timestamp (indexed) |

## daylight_cache_events

Cache operation records.

| Column | Type | Description |
|--------|------|-------------|
| `event_type` | string | `read`, `write`, `delete`, or `exist` (indexed) |
| `key` | string | Cache key |
| `hit` | boolean | Whether a read was a cache hit |
| `duration_ms` | float | Operation time |
| `controller_action` | string | Triggering controller action |
| `request_path` | string | Parent request path |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Timestamp (indexed) |

## daylight_logs

Captured application log entries.

| Column | Type | Description |
|--------|------|-------------|
| `level` | string | `debug`, `info`, `warn`, `error`, `fatal`, `unknown` (indexed) |
| `message` | text | Log message |
| `controller_action` | string | Triggering controller action |
| `request_path` | string | Parent request path |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Timestamp (indexed) |

## daylight_scheduled_tasks

SolidQueue recurring task records.

| Column | Type | Description |
|--------|------|-------------|
| `task_class` | string | Task class name (indexed) |
| `command` | string | Task command |
| `frequency` | string | Execution frequency |
| `status` | string | `completed` or `failed` |
| `duration_ms` | float | Execution time |
| `error_class` | string | Exception class (if failed) |
| `error_message` | text | Exception message (if failed) |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Timestamp (indexed) |

## daylight_mail_events

ActionMailer delivery and processing records.

| Column | Type | Description |
|--------|------|-------------|
| `event_type` | string | `deliver` or `process` (indexed) |
| `mailer_class` | string | Mailer class name (indexed) |
| `action_name` | string | Mailer action |
| `recipients` | text | Recipient addresses |
| `channel` | string | Delivery channel |
| `subject` | string | Email subject |
| `status` | string | `delivered`, `failed`, or `processed` |
| `duration_ms` | float | Delivery/processing time |
| `error_message` | text | Error details (if failed) |
| `trace_id` | string | Distributed trace ID (indexed) |
| `occurred_at` | datetime | Timestamp (indexed) |

## daylight_deploys

Deployment records.

| Column | Type | Description |
|--------|------|-------------|
| `version` | string | Version identifier |
| `description` | text | Deployment description |
| `git_sha` | string | Git commit SHA |
| `deployed_by` | string | Deployer identity |
| `deployed_at` | datetime | Deployment time (indexed) |

## daylight_performance_issues

AI-detected performance issues.

| Column | Type | Description |
|--------|------|-------------|
| `scan_id` | string | Scan batch identifier (indexed) |
| `issue_type` | string | `n_plus_one`, `slow_query`, or `counter_cache` (indexed) |
| `severity` | string | `critical`, `warning`, or `info` |
| `title` | string | Issue title |
| `description` | text | Detailed description |
| `sql_pattern` | string | Normalized SQL causing the issue |
| `source_location` | string | Application code location |
| `controller_action` | string | Affected controller action |
| `occurrences` | integer | Number of times detected |
| `avg_duration_ms` | float | Average query duration |
| `max_duration_ms` | float | Maximum query duration |
| `total_time_ms` | float | Total time across all occurrences |
| `solution` | text | AI-generated fix |
| `status` | string | `open`, `fixed`, or `ignored` (indexed) |
| `detected_at` | datetime | When detected (indexed) |

## daylight_security_issues

Brakeman security scan results.

| Column | Type | Description |
|--------|------|-------------|
| `scan_id` | string | Scan batch identifier (indexed) |
| `issue_type` | string | Category (indexed) |
| `warning_type` | string | Original Brakeman warning type |
| `severity` | string | `critical`, `warning`, or `info` |
| `confidence` | string | `high`, `medium`, or `weak` |
| `title` | string | Issue title |
| `description` | text | Detailed description |
| `file_path` | string | Affected source file |
| `line_number` | integer | Line in source file |
| `code_snippet` | text | Relevant code |
| `check_name` | string | Brakeman check name |
| `link` | string | Brakeman reference URL |
| `fingerprint` | string | Deduplication key (indexed) |
| `solution` | text | AI-generated fix |
| `status` | string | `open`, `fixed`, or `ignored` (indexed) |
| `detected_at` | datetime | When detected (indexed) |

## daylight_incidents

Anomaly-detected incidents.

| Column | Type | Description |
|--------|------|-------------|
| `incident_type` | string | `error_spike`, `latency_spike`, `failure_spike`, `new_error` (indexed) |
| `title` | string | Incident title |
| `summary` | text | AI-generated summary |
| `status` | string | `open`, `investigating`, `resolved`, `false_alarm` |
| `severity` | string | `critical`, `warning`, or `info` |
| `trigger_data` | JSON | Metrics that triggered the incident |
| `investigation` | text | AI investigation report |
| `related_error_id` | integer | Linked error record |
| `related_deploy_id` | integer | Linked deployment |
| `started_at` | datetime | Incident start time |
| `resolved_at` | datetime | When resolved |
| `occurred_at` | datetime | Detection time (indexed) |

## daylight_solutions

AI-generated solution proposals.

| Column | Type | Description |
|--------|------|-------------|
| `source_type` | string | `performance` or `security` |
| `source_issue_id` | integer | Linked issue record |
| `title` | string | Solution title |
| `problem_description` | text | Description of the problem |
| `proposed_fix` | text | Markdown with code changes |
| `file_paths` | JSON | Array of affected file paths |
| `status` | string | `draft`, `approved`, `pushed`, `rejected` |
| `severity` | string | Issue severity |
| `pr_url` | string | GitHub PR URL (after push) |
| `pr_branch` | string | Git branch created |
| `generated_at` | datetime | When generated (indexed) |
| `approved_at` | datetime | When approved |
| `pushed_at` | datetime | When pushed to GitHub |

## daylight_solution_messages

AI chat conversation history for solutions.

| Column | Type | Description |
|--------|------|-------------|
| `solution_id` | integer | Linked solution (indexed) |
| `role` | string | `user` or `assistant` |
| `content` | text | Message content |
| `created_at` | datetime | Message timestamp |

## daylight_settings

Key-value store for dashboard configuration.

| Column | Type | Description |
|--------|------|-------------|
| `key` | string | Setting name (unique, indexed) |
| `value` | text | Setting value |
