---
layout: home
hero:
  name: Daylight
  text: Rails monitoring, simplified.
  tagline: "Drop-in error tracking, performance monitoring, and AI-powered insights for Rails apps — backed by SQLite. No external services required."
  actions:
    - theme: brand
      text: Get Started
      link: /getting-started/
    - theme: alt
      text: View on GitHub
      link: https://github.com/runwell/daylight
features:
  - title: Error Tracking
    details: "Automatic exception capture with fingerprinting, deduplication, and occurrence grouping. Track affected users across your entire stack."
    link: /error-tracking/
  - title: Performance Monitoring
    details: "Request timing, slow query detection, N+1 identification, and Apdex scoring. Full request waterfall with distributed tracing."
    link: /request-monitoring/
  - title: AI-Powered Insights
    details: "Automatic detection of performance bottlenecks and security vulnerabilities with AI-generated fix proposals and one-click GitHub PRs."
    link: /ai-insights/
  - title: Anomaly Detection
    details: "Real-time spike detection for errors, latency, and job failures. AI-investigated incidents with root cause analysis and deploy correlation."
    link: /incidents/
  - title: Background Job Monitoring
    details: "Track ActiveJob execution, failures, and duration across all queues. Monitor SolidQueue scheduled tasks and mail delivery."
    link: /background-jobs/
  - title: Zero Infrastructure
    details: "Everything stored in a local SQLite database. No Redis, no Postgres, no external SaaS. Just add the gem and mount the engine."
    link: /getting-started/
---
