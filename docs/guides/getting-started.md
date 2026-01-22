# Getting Started

This guide will help you set up your development environment and get started with
the projects documented on this site.

## Prerequisites

Before you begin, ensure you have the following installed:

### Required Tools

| Tool | Version | Installation |
|------|---------|--------------|
| Git | Latest | [git-scm.com](https://git-scm.com/) |
| Node.js | 20+ | [nodejs.org](https://nodejs.org/) |
| Python | 3.12+ | [python.org](https://www.python.org/) |
| Go | 1.24+ | [go.dev](https://go.dev/) |
| AWS CLI | 2.x | [AWS CLI Install](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| Terraform | 1.0+ | [terraform.io](https://www.terraform.io/downloads) |

### Optional Tools

| Tool | Purpose | Installation |
|------|---------|--------------|
| Docker | Containerization | [docker.com](https://www.docker.com/) |
| VS Code | IDE | [code.visualstudio.com](https://code.visualstudio.com/) |
| GitHub CLI | GitHub integration | [cli.github.com](https://cli.github.com/) |

## Environment Setup

### 1. Configure Git

```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Enable credential caching
git config --global credential.helper cache
```

### 2. Configure AWS

```bash
# Configure AWS credentials
aws configure

# Verify configuration
aws sts get-caller-identity
```

!!! tip "AWS Profiles"
    Use named profiles for multiple AWS accounts:
    ```bash
    aws configure --profile personal
    aws configure --profile work

    # Use a specific profile
    export AWS_PROFILE=personal
    ```

### 3. Install Node.js Dependencies

```bash
# Install pnpm (recommended)
npm install -g pnpm

# Or use npm
npm install -g npm@latest
```

### 4. Install Python Tools

```bash
# Install uv (fast Python package manager)
pip install uv

# Or use pip
pip install --upgrade pip
```

### 5. Install Go Tools

```bash
# Verify Go installation
go version

# Install common tools
go install golang.org/x/tools/gopls@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

## Project-Specific Setup

### TypeScript Lambda Template

```bash
# Clone the repository
git clone https://github.com/riyanimam/typescript-lambda-template.git
cd typescript-lambda-template

# Install dependencies
pnpm install

# Build
pnpm run build

# Test
pnpm test
```

### Go Lambda Template

```bash
# Clone the repository
git clone https://github.com/riyanimam/go-lambda-template.git
cd go-lambda-template

# Download dependencies
go mod download

# Build
make build

# Test
make test
```

### Python Lambda Template

```bash
# Clone the repository
git clone https://github.com/riyanimam/quick-actions-aws-lambda-template.git
cd quick-actions-aws-lambda-template

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run tests
pytest
```

### Terraform Modules

```bash
# Clone the repository
git clone https://github.com/riyanimam/event-based-terraform-module.git
cd event-based-terraform-module

# Initialize Terraform
cd opentofu
terraform init

# Validate configuration
terraform validate
```

## IDE Configuration

### VS Code Extensions

Recommended extensions for development:

| Extension | Language/Tool |
|-----------|---------------|
| ESLint | JavaScript/TypeScript |
| Prettier | Code formatting |
| Python | Python |
| Go | Go |
| HashiCorp Terraform | Terraform |
| YAML | YAML files |
| Docker | Docker |
| AWS Toolkit | AWS integration |

### VS Code Settings

Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff"
  },
  "[go]": {
    "editor.defaultFormatter": "golang.go"
  },
  "[terraform]": {
    "editor.defaultFormatter": "hashicorp.terraform"
  },
  "python.linting.enabled": true,
  "go.lintTool": "golangci-lint"
}
```

## Verification

Run these commands to verify your setup:

```bash
# Git
git --version

# Node.js
node --version
pnpm --version

# Python
python --version
pip --version

# Go
go version

# AWS
aws --version
aws sts get-caller-identity

# Terraform
terraform --version
```

Expected output:

```text
git version 2.40+
v20.x.x
8.x.x
Python 3.12.x
pip 24.x
go version go1.24+
aws-cli/2.x.x
{
    "UserId": "...",
    "Account": "...",
    "Arn": "..."
}
Terraform v1.x.x
```

## Common Issues

### Node.js Version Issues

Use a version manager:

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install and use Node.js 20
nvm install 20
nvm use 20
```

### Python Virtual Environment Issues

```bash
# If venv fails, install python-venv
sudo apt install python3-venv  # Ubuntu/Debian

# Or use pyenv
curl https://pyenv.run | bash
```

### AWS Credential Issues

```bash
# Check current credentials
aws configure list

# Reset credentials
rm -rf ~/.aws/credentials
aws configure
```

### Go Module Issues

```bash
# Clear module cache
go clean -modcache

# Download modules again
go mod download
```

## Next Steps

Now that your environment is set up:

1. **Explore the projects** - Browse the [Projects](../projects/index.md) section
2. **Learn workflows** - Read [Development Workflows](development-workflows.md)
3. **Set up CI/CD** - Follow [CI/CD Best Practices](cicd-best-practices.md)
4. **Review security** - Check [Security Guidelines](security-guidelines.md)

## Getting Help

If you encounter issues:

- Check the [Troubleshooting](../troubleshooting/index.md) section
- Open an issue on the relevant GitHub repository
- Review the project-specific documentation
