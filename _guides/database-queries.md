---
title: Database Queries
nav_order: 4
description: Track slow database queries, detect N+1 patterns, and identify counter cache opportunities.
---

Daylight monitors your application's database queries to surface slow queries and problematic patterns.

## Slow query tracking

By default, Daylight records any SQL query that takes 50ms or longer. Faster queries are counted for per-request statistics but not individually stored, keeping database growth manageable.

The slow query threshold is configurable in the dashboard Settings page.

For each slow query, Daylight captures:

- The full SQL statement
- A normalized version (with literals replaced by `?`) for grouping
- Execution duration
- Source location in your application code (e.g., `app/models/user.rb:42`)
- The controller action and request path that triggered it
- The associated request and trace IDs

## Query normalization

SQL statements are normalized by replacing literal values with placeholders:

```sql
-- Raw SQL
SELECT * FROM users WHERE id = 42 AND active = true

-- Normalized
SELECT * FROM users WHERE id = ? AND active = ?
```

This allows Daylight to group identical query patterns regardless of the specific parameter values, so you can see how often a particular query shape runs and its average performance.

## N+1 query detection

Daylight detects N+1 patterns by tracking normalized SQL statements within each request trace. When the same normalized query executes 3 or more times in a single request, it is flagged as a potential N+1.

The queries dashboard groups these by pattern, showing:

- The normalized SQL
- How many times it executes per request
- Total time spent across all executions
- The controller action where it occurs

## Counter cache candidates

Daylight identifies `COUNT(*)` queries that execute 10 or more times in a 24-hour window. These are candidates for a Rails `counter_cache` column, which eliminates the count queries entirely:

```ruby
class Comment < ApplicationRecord
  belongs_to :post, counter_cache: true
end
```

## Linking queries to requests

Every query is linked to its parent request via `trace_id`. Click through from the queries dashboard to see the full request waterfall, or drill down from a request to see all its queries.
