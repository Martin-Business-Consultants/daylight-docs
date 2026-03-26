---
title: Deploys
nav_order: 9
description: Track deployments to correlate performance changes and incidents with specific releases.
---

Recording deployments lets Daylight overlay deploy markers on performance charts and correlate incidents with specific releases.

## Recording a deployment

Call `Daylight.deploy!` in your deployment pipeline:

```ruby
Daylight.deploy!(
  version: "1.2.3",
  description: "Fix N+1 in user list",
  git_sha: "abc123def456",
  deployed_by: "alice@example.com"
)
```

All parameters are optional — pass whatever metadata is available in your deployment context.

### Capistrano example

```ruby
after "deploy:finished", "daylight:record" do
  on primary(:app) do
    within release_path do
      execute :rails, "runner", "'Daylight.deploy!(version: \"#{fetch(:release_timestamp)}\", git_sha: \"#{fetch(:current_revision)}\")'"
    end
  end
end
```
{: data-title="config/deploy.rb"}

### Kamal example

```ruby
# In a post-deploy hook
Daylight.deploy!(
  version: ENV["KAMAL_VERSION"],
  git_sha: `git rev-parse HEAD`.strip,
  deployed_by: ENV["USER"]
)
```

## Dashboard integration

Deploy markers appear on:

- **Request latency charts** — see if latency changed after a deploy
- **Throughput charts** — spot traffic shifts around releases
- **Incident details** — when a deploy is correlated with an incident

## Incident correlation

When an incident is detected within 30 minutes of a deployment, Daylight automatically links them. If GitHub integration is configured, the git diff between deploys is included in the AI investigation context to help identify the root cause.
