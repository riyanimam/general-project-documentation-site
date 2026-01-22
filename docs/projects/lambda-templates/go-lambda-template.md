# Go Lambda Template

[![Go Version](https://img.shields.io/badge/Go-1.24+-00ADD8?logo=go)](https://go.dev/)
[![AWS Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?logo=amazon-aws)](https://aws.amazon.com/lambda/)
[![golangci-lint](https://img.shields.io/badge/golangci--lint-latest-blue)](https://golangci-lint.run/)

A production-ready Go template for AWS Lambda functions with comprehensive tooling, testing, and
CI/CD pipelines. Optimized for minimal cold starts and high performance.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/go-lambda-template){ .md-button }

## Features

- 🚀 **AWS Lambda Ready**: Pre-configured for AWS Lambda deployment with custom runtime
- 🛠️ **Modern Tooling**: golangci-lint, gofmt, lefthook for code quality
- 🧪 **Testing**: Built-in test infrastructure with coverage reporting
- 📦 **CI/CD**: GitHub Actions workflows for testing, security scanning, and deployment
- 🔒 **Security**: Integrated gosec, govulncheck, and CodeQL scanning
- 📝 **Documentation**: Comprehensive development and deployment guides

## Prerequisites

- **Go**: 1.24 or higher
- **Git**: 2.30 or higher
- **AWS CLI**: Configured with appropriate credentials
- **Terraform**: 1.10 or higher (for infrastructure deployment)

## Quick Start

```bash
# Clone the repository
git clone https://github.com/riyanimam/go-lambda-template.git
cd go-lambda-template

# Install dependencies
go mod download

# Install development tools
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install github.com/evilmartians/lefthook@latest
go install golang.org/x/vuln/cmd/govulncheck@latest

# Install git hooks
lefthook install

# Run tests
go test -v ./...
```

## Project Structure

```text
.
├── .github/
│   └── workflows/          # GitHub Actions CI/CD workflows
├── src/
│   ├── main.go            # Lambda handler implementation
│   └── main_test.go       # Handler tests
├── terraform/             # Infrastructure as Code
├── .golangci.yml          # golangci-lint configuration
├── lefthook.yml           # Git hooks configuration
├── go.mod                 # Go module dependencies
└── README.md
```

## Lambda Handler

The handler (`src/main.go`) processes incoming requests:

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/aws/aws-lambda-go/lambda"
)

type Request struct {
    Name string `json:"name,omitempty"`
}

type Response struct {
    Message string `json:"message"`
    Status  int    `json:"status"`
}

func handler(_ context.Context, request Request) (Response, error) {
    name := "world"
    if request.Name != "" {
        name = request.Name
    }

    return Response{
        Message: fmt.Sprintf("Hello, %s!", name),
        Status:  200,
    }, nil
}

func main() {
    lambda.Start(handler)
}
```

## Building for AWS Lambda

### Local Build

```bash
# Standard build
go build -o bootstrap ./src/main.go

# Build with optimizations (smaller binary)
go build -ldflags="-s -w" -o bootstrap ./src/main.go
```

### Production Build

```bash
# Build for Linux AMD64 (AWS Lambda runtime)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build \
  -tags lambda.norpc \
  -ldflags="-s -w" \
  -o bootstrap ./src/main.go

# Create deployment package
zip lambda-function.zip bootstrap
```

**Build flags explained:**

| Flag | Purpose |
|------|---------|
| `GOOS=linux GOARCH=amd64` | Target Linux AMD64 architecture |
| `CGO_ENABLED=0` | Disable CGO for static binary |
| `-tags lambda.norpc` | Use optimized Lambda runtime |
| `-ldflags="-s -w"` | Strip debug info (reduces size) |

## Development Workflow

### Code Quality

```bash
# Format all Go files
gofmt -l -s -w .

# Run golangci-lint with all configured linters
golangci-lint run ./...

# Run with auto-fix where possible
golangci-lint run --fix ./...

# Run go vet
go vet ./...

# Check for vulnerabilities
govulncheck ./...
```

### Testing

```bash
# Run all tests
go test -v ./...

# Run tests with coverage
go test -v -race -coverprofile=coverage.out ./...

# View coverage report in terminal
go tool cover -func=coverage.out

# Generate HTML coverage report
go tool cover -html=coverage.out -o coverage.html
```

### Git Hooks

**Pre-commit:**

- Code formatting (`gofmt`)
- Static analysis (`go vet`)
- Terraform formatting
- YAML linting
- Markdown formatting

**Pre-push:**

- Full test suite with race detection
- Build verification

```bash
# Run checks manually
lefthook run pre-commit
lefthook run pre-push
```

## Terraform Deployment

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

### Terraform Configuration

```hcl
resource "aws_lambda_function" "go_lambda" {
  function_name = "go_lambda_handler"
  filename      = "build/go_lambda.zip"
  handler       = "main"
  runtime       = "go1.x"
  role          = aws_iam_role.lambda_exec.arn

  environment {
    variables = {
      ENV = "production"
    }
  }

  tags = {
    Project     = "GoLambdaTemplate"
    Environment = "Production"
  }
}
```

## CI/CD Workflows

GitHub Actions workflows include:

- **CI**: Multi-version testing (Go 1.23.x, 1.24.x) and artifact building
- **Code Quality**: Linting, formatting checks, and test coverage
- **Security**: gosec, govulncheck, and CodeQL scanning
- **PR Validation**: Conventional commit message validation

## Gotchas & Tips

!!! warning "Security Vulnerabilities in Go Standard Library"
    Go 1.24 includes critical security fixes. Always use the latest stable version.
    The template was updated to fix 9 vulnerabilities in crypto/x509, net/http, etc.

!!! tip "Cold Start Optimization"
    Use `-tags lambda.norpc` flag to reduce binary size and improve cold start times.
    Combined with `-ldflags="-s -w"`, this can reduce binary size by 30-40%.

!!! note "Race Condition Detection"
    Always run tests with `-race` flag in CI to detect race conditions early.
    This is automatically enabled in the pre-push hooks.

## Troubleshooting

### Module Download Failures

```bash
# Clear module cache
go clean -modcache

# Re-download dependencies
go mod download
```

### Build Failures on Lambda

Ensure you're building for the correct architecture:

```bash
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o bootstrap ./src/main.go
```

### golangci-lint Issues

```bash
# Update to latest version
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Clear cache
golangci-lint cache clean
```

### Lefthook Not Running

```bash
# Reinstall hooks
lefthook uninstall
lefthook install
```

## Enhancement Ideas

- [ ] Add structured logging with zerolog or zap
- [ ] Implement graceful shutdown handling
- [ ] Add OpenTelemetry tracing
- [ ] Support ARM64 architecture (Graviton2)
- [ ] Add API Gateway integration example
- [ ] Implement DynamoDB integration

## Best Practices

1. **Always run tests before pushing**
2. **Keep functions small and focused**
3. **Use meaningful variable and function names**
4. **Document exported functions and types**
5. **Handle errors explicitly**
6. **Use contexts for cancellation and timeouts**
7. **Keep dependencies minimal**
8. **Review security scan results**

## Resources

- [Effective Go](https://go.dev/doc/effective_go)
- [AWS Lambda Go Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/golang-handler.html)
- [golangci-lint Documentation](https://golangci-lint.run/)
- [Conventional Commits](https://www.conventionalcommits.org/)
