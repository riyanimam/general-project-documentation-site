# TypeScript Lambda Template

[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-20+-green?logo=node.js)](https://nodejs.org/)
[![AWS Lambda](https://img.shields.io/badge/AWS-Lambda-orange?logo=amazon-aws)](https://aws.amazon.com/lambda/)

A production-ready AWS Lambda template using TypeScript with ECMAScript Modules (ESM), featuring
S3-to-PostgreSQL CSV processing, comprehensive testing, and automated code quality workflows.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/typescript-lambda-template){ .md-button }

## Features

- **TypeScript with ESM**: Native ES modules using `.mts` extension and `NodeNext` resolution
- **AWS SDK v3**: Modern AWS integrations with S3, SQS event handling
- **PostgreSQL Integration**: Batch CSV ingestion from S3 to PostgreSQL
- **Production-Ready**: Error handling, retries, configurable batch processing
- **Code Quality**: Biome for linting/formatting, Lefthook for git hooks
- **Type Safety**: Full TypeScript types with AWS Lambda event definitions
- **Testing**: Vitest with native ESM support and AWS SDK mocking

## Prerequisites

- **Node.js** >= 20 (Lambda runtime: `nodejs20.x` or `nodejs22.x`)
- **pnpm** >= 10 (fast, disk-efficient package manager)
- **PostgreSQL** (for local development and testing)
- **AWS Account** (for deployment)

## Quick Start

```bash
# Clone the repository
git clone https://github.com/riyanimam/typescript-lambda-template.git
cd typescript-lambda-template

# Install dependencies
pnpm install

# Build the project
pnpm run build

# Run tests
pnpm test
```

## Project Structure

```text
typescript-lambda-example/
├── src/
│   ├── handler.mts          # Main Lambda handler for SQS/S3 events
│   └── s3-to-mssql.mts      # S3 CSV to PostgreSQL pipeline
├── tests/
│   └── handler.test.mts     # Vitest tests with AWS SDK mocks
├── .github/workflows/       # CI/CD automation
├── dist/                    # Compiled JavaScript output
├── tsconfig.json            # TypeScript ESM configuration
├── vitest.config.mts        # Vitest configuration
├── biome.json               # Biome linter/formatter config
├── lefthook.yml             # Git hooks configuration
└── package.json             # Dependencies and scripts
```

## Lambda Handler

The main handler processes SQS events containing S3 event notifications:

1. Parses SQS messages and extracts S3 event details
2. Fetches CSV files from S3
3. Streams and parses CSV data
4. Batch inserts rows into PostgreSQL
5. Handles errors with configurable retry logic

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PGHOST` | PostgreSQL host | - |
| `PGPORT` | PostgreSQL port | 5432 |
| `PGDATABASE` | Database name | - |
| `PGUSER` | Database user | - |
| `PGPASSWORD` | Database password | - |
| `TABLE_NAME` | Target table | `csv_raw_rows` |
| `BATCH_SIZE` | Insert batch size | 100 |
| `THROW_ON_ERROR` | Throw on DB errors for SQS retry | `true` |

## Development Workflow

### Code Quality with Biome

```bash
# Check for issues (linting + formatting)
pnpm run lint

# Auto-fix issues
pnpm run lint:fix

# Format code only
pnpm run format
```

### Running Tests

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm run test:watch

# Run tests with coverage
pnpm run test:coverage
```

### Git Hooks with Lefthook

Git hooks are automatically installed during `pnpm install`:

**Pre-commit:**

- Run Biome linter and formatter on staged files
- Lint markdown files
- Remove trailing whitespace

**Pre-push:**

- Run tests
- Build the project

## Deployment

### Package for Lambda

=== "Linux/macOS"

    ```bash
    pnpm run build
    zip -r lambda-package.zip dist node_modules package.json
    ```

=== "Windows PowerShell"

    ```powershell
    pnpm run build
    Compress-Archive -Path dist, node_modules, package.json `
      -DestinationPath lambda-package.zip
    ```

### Terraform Configuration

```hcl
resource "aws_lambda_function" "csv_processor" {
  filename         = "lambda-package.zip"
  function_name    = "s3-csv-to-postgres"
  role            = aws_iam_role.lambda_role.arn
  handler         = "dist/handler.handler"
  runtime         = "nodejs20.x"
  timeout         = 300
  memory_size     = 512

  environment {
    variables = {
      PGHOST         = var.db_host
      PGDATABASE     = var.db_name
      TABLE_NAME     = "csv_data"
      BATCH_SIZE     = "100"
    }
  }
}
```

## TypeScript and ESM

This project uses **ECMAScript Modules** with TypeScript:

- Source files use `.mts` extension
- `package.json` sets `"type": "module"`
- `tsconfig.json` uses `"module": "NodeNext"`
- Local imports require explicit `.mjs` extensions

### Example Import

```typescript
import type { SQSEvent } from "aws-lambda";
import { S3Client } from "@aws-sdk/client-s3";
import { myFunction } from "./utils.mjs"; // Note .mjs extension
```

## Gotchas & Tips

!!! warning "ESM Import Extensions"
    Always use `.mjs` extension for local imports in TypeScript ESM.
    The compiled output uses `.mjs`, so imports must match.

!!! tip "Cold Start Optimization"
    Initialize AWS clients and database pools outside the handler function
    to reuse connections across invocations.

!!! note "SQS Retry Logic"
    Set `THROW_ON_ERROR=true` to allow SQS to retry failed messages.
    The function will throw an error, and SQS will handle retries.

## Troubleshooting

### ESM Import Issues

If you see `ERR_MODULE_NOT_FOUND`:

- Ensure local imports use `.mjs` extension
- Check `package.json` has `"type": "module"`
- Verify `tsconfig.json` uses `NodeNext` module resolution

### Vitest Test Failures

- Check `vitest.config.mts` for test-specific configurations
- Use `pnpm run test:watch` for debugging with watch mode
- Vitest runs natively with ESM, no experimental flags needed

### Lefthook Hook Failures

```bash
# Run pre-commit checks manually
npx lefthook run pre-commit

# Auto-fix with
pnpm run lint:fix && pnpm run format

# Reinstall hooks
npx lefthook install
```

## Enhancement Ideas

- [ ] Add support for Parquet file processing
- [ ] Implement dead-letter queue handling
- [ ] Add CloudWatch metrics for batch processing
- [ ] Support multiple database backends (MySQL, DynamoDB)
- [ ] Add streaming response for large files

## Resources

- [AWS Lambda TypeScript](https://docs.aws.amazon.com/lambda/latest/dg/lambda-typescript.html)
- [TypeScript ESM Documentation](https://www.typescriptlang.org/docs/handbook/esm-node.html)
- [AWS SDK for JavaScript v3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/)
- [Vitest Documentation](https://vitest.dev/)
- [Biome Documentation](https://biomejs.dev/)
