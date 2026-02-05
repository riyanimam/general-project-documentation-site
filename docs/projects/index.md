# Projects Overview

This documentation covers all public projects organized by category. Each project is
production-ready with comprehensive CI/CD, testing, and documentation.

## Categories

### Lambda Templates

Production-ready AWS Lambda boilerplates for different languages:

| Project | Language | Key Features |
|---------|----------|--------------|
| [TypeScript Lambda Template](lambda-templates/typescript-lambda-template.md) | TypeScript | ESM, Biome, Vitest, PostgreSQL |
| [Go Lambda Template](lambda-templates/go-lambda-template.md) | Go | Minimal cold starts, golangci-lint |
| [Quick Actions Lambda](lambda-templates/quick-actions-aws-lambda-template.md) | Python | Multi-service, Ruff, SQS |

### Terraform Modules

Reusable Infrastructure as Code modules:

| Module | Event Sources | Key Features |
|--------|---------------|--------------|
| [Event-Based Module](terraform-modules/event-based-terraform-module.md) | SQS, DynamoDB, Kinesis, EventBridge | Security scanning, examples |

### Bot Playground

Messaging bot development and experimentation:

| Project | Platform | Stack |
|---------|----------|-------|
| [Discord Bot Playground](bot-playground/discord-bot-playground.md) | Discord | Go, DiscordGo |
| [Signal Bot Playground](bot-playground/signal-bot-playground.md) | Signal | Go, Signal CLI |

### DevOps Tools

Repository management and CI/CD automation:

| Project | Purpose | Stack |
|---------|---------|-------|
| [Common Repo Assets](devops-tools/common-repo-assets.md) | GitHub rulesets & configs | JSON, YAML |
| [Automated GitHub to New Relic Synthetics](devops-tools/automated-github-to-new-relic-synthetics.md) | Synthetics automation | JavaScript, GitHub Actions |

### Frontend Projects

User interface and web applications:

| Project | Purpose | Stack |
|---------|---------|-------|
| [Shotgun Socials Poster](frontend/shotgun-socials-poster.md) | Multi-platform social poster | React, TypeScript, Vite |

### Utility Projects

Developer tools and automation:

| Project | Purpose | Stack |
|---------|---------|-------|
| [AWS Cleanup Lambda](utilities/aws-environment-cleanup-lambda.md) | Clean unused AWS resources | Python, Terraform |
| [Georgia Legislation Webcrawler](utilities/georgia-legislation-webcrawler.md) | Scrape legislation data | React, Python, Playwright |
| [Kanban Board Template](utilities/not-confusing-kanban-board-template.md) | GitHub project template | Markdown |

## Common Patterns

All projects share common patterns for consistency:

### Code Quality

```yaml
# Pre-commit / Lefthook hooks
- Linting (language-specific)
- Formatting
- Type checking
- Commit message validation
```

### CI/CD Workflows

```yaml
# GitHub Actions
- Build and test
- Security scanning
- Terraform validation
- Semantic release
```

### Documentation

```text
project/
├── README.md           # Quick start
├── DEVELOPMENT.md      # Developer guide
├── CHANGELOG.md        # Version history
├── CONTRIBUTING.md     # Contribution guidelines
└── SECURITY.md         # Security policies
```

## Getting Started

1. **Choose a project** based on your use case
2. **Read the README** for quick start instructions
3. **Review DEVELOPMENT.md** for detailed setup
4. **Check TROUBLESHOOTING** section for common issues
