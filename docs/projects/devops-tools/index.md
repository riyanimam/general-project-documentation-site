# DevOps Tools

This section covers DevOps and automation tools for repository management and CI/CD workflows.

## Project Overview

| Project | Description | Stack | Primary Use |
|---------|-------------|-------|-------------|
| [Common Repo Assets](common-repo-assets.md) | Reusable GitHub rulesets & configs | JSON/YAML | Repository governance |
| [Automated GitHub to New Relic Synthetics](automated-github-to-new-relic-synthetics.md) | CI/CD for synthetic monitors | JavaScript/GitHub Actions | Observability automation |

## Categories

### Repository Management

Tools for managing GitHub repositories at scale.

<div class="grid cards" markdown>

- :material-shield-check:{ .lg .middle } **Common Repo Assets**

    ---

    A collection of reusable GitHub repository rulesets and branch protection
    configurations that help enforce best practices.

    [:octicons-arrow-right-24: Learn more](common-repo-assets.md)

</div>

### Observability Automation

Projects for automating monitoring and observability workflows.

<div class="grid cards" markdown>

- :material-chart-timeline:{ .lg .middle } **Automated GitHub to New Relic Synthetics**

    ---

    Automates syncing and managing New Relic Synthetics monitors from changes
    in a GitHub repository.

    [:octicons-arrow-right-24: Learn more](automated-github-to-new-relic-synthetics.md)

</div>

## Common Patterns

### GitHub Actions Best Practices

All CI/CD workflows follow consistent patterns:

```yaml
name: Workflow Name

on:
  push:
    branches: [main]
  pull_request:

jobs:
  job-name:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Step name
        run: |
          echo "Action here"
```

### Configuration as Code

Consistent YAML/JSON configuration:

```yaml
# Repository ruleset example
name: main-protection
enforcement: active
target: branch
conditions:
  ref_name:
    include:
      - "refs/heads/main"
rules:
  - type: pull_request
    parameters:
      required_approving_review_count: 2
      dismiss_stale_reviews_on_push: true
```

## Quick Links

- [Common Repo Assets :material-arrow-right:](common-repo-assets.md)
- [Automated GitHub to New Relic Synthetics :material-arrow-right:](automated-github-to-new-relic-synthetics.md)
