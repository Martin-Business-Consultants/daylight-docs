---
title: AI-Powered Insights
nav_order: 6
description: Detect performance bottlenecks and security vulnerabilities with AI-generated solutions and one-click GitHub PRs.
---

Daylight uses AI to analyze your application for performance issues and security vulnerabilities, then generates actionable fix proposals that you can refine and push directly as GitHub pull requests.

## Setup

Configure an AI provider in the dashboard Settings page. Daylight supports:

| Provider | Models |
|----------|--------|
| **Google Gemini** | `gemini-2.5-flash`, `gemini-2.5-pro` |
| **Anthropic Claude** | `claude-haiku-4-5-20251001`, `claude-sonnet-4-6`, `claude-opus-4-6` |

Enter your API key in Settings and select a default model. Daylight uses the [ruby_llm](https://github.com/crmne/ruby_llm) gem to communicate with both providers.

{% include alert.html type="tip" content="Start with <code>gemini-2.5-flash</code> or <code>claude-haiku-4-5-20251001</code> for fast, cost-effective scanning. Use larger models for complex security issues." %}

## Performance scanning

The performance scanner analyzes your recorded query data from the last 24 hours to detect:

### N+1 queries

Groups queries by normalized SQL per request trace. An issue is flagged when the same query repeats 3+ times per request.

| Severity | Criteria |
|----------|----------|
| **Critical** | >50 executions per request, or >5s total time in 24h |
| **Warning** | >10 executions per request, or >1s total time in 24h |
| **Info** | 3+ executions per request |

### Slow query patterns

Identifies normalized SQL patterns with average duration above the threshold (default 50ms). Severity is based on average and maximum duration.

### Counter cache candidates

Detects `COUNT(*)` queries executing 10+ times in 24 hours and suggests adding `counter_cache` columns.

### Running a scan

Scans run on a configurable schedule (every 6 hours, 12 hours, daily, or weekly) or on-demand from the Settings page. Results appear in the Settings page under Performance Issues.

## Security scanning

Daylight integrates [Brakeman](https://brakemanscanner.org/) for static security analysis of your Rails application.

### Issues detected

- SQL injection
- Cross-site scripting (XSS)
- CSRF vulnerabilities
- Mass assignment
- Remote code execution
- Unsafe redirects
- File access / path traversal
- SSL/TLS misconfiguration
- Authentication and session issues
- Unsafe rendering

### Confidence and severity mapping

| Brakeman confidence | Daylight severity |
|---------------------|-------------------|
| High | Critical |
| Medium | Warning |
| Weak | Info |

Issues are deduplicated by Brakeman's internal fingerprint — the same vulnerability won't generate duplicate entries across scans.

## AI solutions

For each detected performance or security issue, Daylight generates an AI solution that includes:

1. **Root cause explanation** — why this is a problem
2. **Proposed code fix** — before/after code with file paths
3. **Alternative approaches** — other ways to address the issue

### Refining solutions

Each solution has a built-in AI chat interface. Send messages to refine the proposal:

- "Use `includes` instead of `preload` here"
- "This model uses STI, update the fix accordingly"
- "Also add an index for this query"

The AI maintains conversation history and updates the proposed fix based on your feedback.

### Regenerating solutions

If a solution misses the mark, click **Regenerate** to start fresh with a new AI analysis.

## GitHub integration

Daylight can push approved solutions directly as GitHub pull requests.

### Setup

In the dashboard Settings page, configure:

- **GitHub repo URL** — your repository (e.g., `https://github.com/org/repo`)
- **Default branch** — the branch PRs target (e.g., `main`)
- **GitHub API token** — a personal access token with `repo` scope

### Pushing a solution

1. Review the AI-generated solution
2. Refine via chat if needed
3. Click **Push to GitHub**

Daylight will:

- Create a branch named `daylight/solution-{id}-{timestamp}`
- Commit the proposed file changes
- Open a pull request against your default branch
- Link the PR back to the solution in the dashboard

## Live N+1 diagnostics with Bullet

Daylight integrates the [Bullet](https://github.com/flyerhzm/bullet) gem for live N+1 detection during actual request execution.

### How it works

- **Development**: Bullet is always active
- **Production**: Bullet runs during time-limited diagnostic windows with 5% request sampling

### Starting a diagnostic window

From the Settings page, click **Toggle Bullet Diagnostic** and set a duration (5–120 minutes). During this window, Bullet instruments a sample of production requests and reports any N+1 queries, unused eager loads, or missing counter caches it finds.

{% include alert.html type="warning" content="Bullet adds overhead to instrumented requests. Use short diagnostic windows in production and rely on the sampling rate to limit impact." %}
