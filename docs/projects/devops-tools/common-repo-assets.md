# Common Repo Assets

[![Configuration](https://img.shields.io/badge/Type-Configuration-blue)](https://github.com/riyanimam/common-repo-assets)
[![GitHub](https://img.shields.io/badge/Platform-GitHub-181717?logo=github)](https://github.com)
[![Security](https://img.shields.io/badge/Focus-Security-green)](https://github.com/riyanimam/common-repo-assets)

A collection of reusable GitHub repository rulesets and branch protection configurations
that help enforce best practices, secure workflows, and consistent structures across
different project types. Includes examples, JSON schemas, and customizable protection
presets to save time and standardize governance.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/common-repo-assets){ .md-button }

## Features

- **Repository Rulesets**: Pre-configured ruleset templates
- **Branch Protection**: Ready-to-use branch protection configs
- **Security Policies**: Standard security configurations
- **Code Review Rules**: Enforce code review best practices
- **CI/CD Integration**: Workflow status check requirements
- **Customizable**: Easy to adapt to your needs

## Overview

Managing multiple repositories with consistent governance can be challenging. This
project provides battle-tested configuration templates that you can reuse across
all your repositories.

## Repository Rulesets

### Main Branch Protection

```json
{
  "name": "main-branch-protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["refs/heads/main"],
      "exclude": []
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 2,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true,
        "require_last_push_approval": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [
          {
            "context": "build",
            "integration_id": null
          },
          {
            "context": "test",
            "integration_id": null
          },
          {
            "context": "lint",
            "integration_id": null
          }
        ]
      }
    },
    {
      "type": "deletion"
    },
    {
      "type": "non_fast_forward"
    },
    {
      "type": "required_signatures"
    }
  ]
}
```

### Development Branch

```json
{
  "name": "dev-branch-protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["refs/heads/dev", "refs/heads/develop"],
      "exclude": []
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": false,
        "required_status_checks": [
          {
            "context": "build"
          },
          {
            "context": "test"
          }
        ]
      }
    }
  ]
}
```

## Quick Start

### Using the Rulesets

1. **Clone the repository**:

   ```bash
   git clone https://github.com/riyanimam/common-repo-assets.git
   cd common-repo-assets
   ```

2. **Choose a ruleset** from the `rulesets/` directory

3. **Apply via GitHub UI**:
   - Navigate to your repository → Settings → Rules → Rulesets
   - Click "New ruleset"
   - Import or copy the JSON configuration

4. **Apply via GitHub CLI**:

   ```bash
   gh api repos/{owner}/{repo}/rulesets \
     -X POST \
     --input rulesets/main-branch-protection.json
   ```

### Using the GitHub API

```bash
# Create ruleset
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/{owner}/{repo}/rulesets \
  -d @rulesets/main-branch-protection.json

# List existing rulesets
gh api repos/{owner}/{repo}/rulesets

# Update ruleset
gh api repos/{owner}/{repo}/rulesets/{ruleset_id} \
  -X PUT \
  --input rulesets/updated-config.json
```

## Available Configurations

### Ruleset Templates

| Template | Use Case | Strictness |
|----------|----------|------------|
| `main-production.json` | Production main branch | Very High |
| `main-standard.json` | Standard main branch | High |
| `develop.json` | Development branch | Medium |
| `feature.json` | Feature branches | Low |
| `hotfix.json` | Hotfix branches | Medium-High |

### Security Configurations

| Configuration | Purpose |
|---------------|---------|
| `require-signed-commits.json` | Enforce commit signing |
| `require-2fa.json` | Require 2FA for contributors |
| `security-scanning.json` | Enable security scanning |
| `dependabot.json` | Automated dependency updates |

### Code Review Templates

| Template | Review Requirements |
|----------|---------------------|
| `strict-review.json` | 2+ approvals, code owners |
| `standard-review.json` | 1+ approval |
| `light-review.json` | Optional reviews |

## Project Structure

```
common-repo-assets/
├── rulesets/
│   ├── branch-protection/
│   │   ├── main-production.json
│   │   ├── main-standard.json
│   │   ├── develop.json
│   │   └── feature.json
│   ├── security/
│   │   ├── signed-commits.json
│   │   ├── 2fa-required.json
│   │   └── security-scanning.json
│   └── code-review/
│       ├── strict-review.json
│       ├── standard-review.json
│       └── light-review.json
├── workflows/
│   ├── apply-rulesets.yml
│   └── validate-rulesets.yml
├── scripts/
│   ├── apply-to-org.sh
│   └── validate-config.sh
├── docs/
│   └── ruleset-guide.md
└── README.md
```

## Customization

### Modify Review Requirements

```json
{
  "type": "pull_request",
  "parameters": {
    "required_approving_review_count": 1,  // Change this
    "dismiss_stale_reviews_on_push": true,
    "require_code_owner_review": false     // Or this
  }
}
```

### Add Status Checks

```json
{
  "type": "required_status_checks",
  "parameters": {
    "required_status_checks": [
      {"context": "build"},
      {"context": "test"},
      {"context": "security-scan"},  // Add your checks
      {"context": "lint"},
      {"context": "coverage"}
    ]
  }
}
```

### Branch Name Patterns

```json
{
  "conditions": {
    "ref_name": {
      "include": [
        "refs/heads/main",
        "refs/heads/release/*",  // Wildcard patterns
        "refs/heads/hotfix/*"
      ],
      "exclude": [
        "refs/heads/temp/*"      // Exclude patterns
      ]
    }
  }
}
```

## Automation

### Apply to Multiple Repositories

```bash
#!/bin/bash
# apply-to-org.sh

ORG="your-org"
RULESET_FILE="rulesets/main-standard.json"

for repo in $(gh repo list $ORG --json name -q '.[].name'); do
  echo "Applying ruleset to $repo..."
  gh api repos/$ORG/$repo/rulesets \
    -X POST \
    --input $RULESET_FILE
done
```

### GitHub Actions Workflow

```yaml
name: Apply Rulesets

on:
  push:
    paths:
      - 'rulesets/**'

jobs:
  apply:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Apply rulesets
        env:
          GH_TOKEN: ${{ secrets.PAT_TOKEN }}
        run: |
          for file in rulesets/branch-protection/*.json; do
            gh api repos/${{ github.repository }}/rulesets \
              -X POST \
              --input "$file"
          done
```

## Best Practices

!!! tip "Start Conservative"
    Begin with stricter rules and relax them as needed. It's easier to remove
    restrictions than to add them later.

!!! warning "Test First"
    Apply rulesets to a test repository first to ensure they work as expected
    before rolling out organization-wide.

!!! note "Document Exceptions"
    If you need to bypass rules, document why and use bypass lists sparingly:
    ```json
    {
      "bypass_actors": [
        {
          "actor_id": 1,
          "actor_type": "Team",
          "bypass_mode": "always"
        }
      ]
    }
    ```

!!! danger "Avoid Over-Restriction"
    Don't make rules so strict that they block legitimate work. Balance security
    with developer productivity.

## Gotchas & Tips

!!! warning "API Token Permissions"
    You need admin access to the repository to create/modify rulesets.
    Personal Access Tokens (PAT) must have `admin:org` scope.

!!! tip "Ruleset Priority"
    When multiple rulesets apply, GitHub uses the most restrictive rules.
    Order matters!

!!! note "Enforcement Levels"
    - `active`: Rules are enforced
    - `evaluate`: Rules are checked but not enforced (testing mode)
    - `disabled`: Rules are not applied

## Troubleshooting

### Ruleset Not Applying

1. **Check Enforcement**: Ensure enforcement is set to `active`
2. **Verify Branch Pattern**: Check the ref_name matches your branches
3. **Check Permissions**: Verify you have admin access

### Status Checks Failing

```bash
# List required status checks
gh api repos/{owner}/{repo}/rulesets/{id} | jq '.rules[] | select(.type=="required_status_checks")'

# Verify workflow runs
gh run list --repo {owner}/{repo}
```

### Bypass Not Working

```json
// Correct bypass configuration
{
  "bypass_actors": [
    {
      "actor_id": 5,              // Team/User ID
      "actor_type": "Team",       // or "RepositoryRole", "OrganizationAdmin"
      "bypass_mode": "always"     // or "pull_request"
    }
  ]
}
```

## Enhancement Ideas

- [ ] Add Terraform modules for ruleset management
- [ ] Create CLI tool for easier application
- [ ] Add validation scripts
- [ ] Create ruleset comparison tool
- [ ] Add organization-wide templates
- [ ] Create migration scripts from legacy branch protection
- [ ] Add monitoring/compliance dashboard
- [ ] Implement ruleset versioning

## Resources

- [GitHub Rulesets Documentation](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [GitHub Branch Protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub REST API - Rulesets](https://docs.github.com/en/rest/repos/rules)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
