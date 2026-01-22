# Terraform Modules

Reusable Infrastructure as Code modules for event-driven AWS architectures. Built with security
scanning, comprehensive documentation, and production-ready examples.

## Available Modules

| Module | Description | Event Sources |
|--------|-------------|---------------|
| [Event-Based Module](event-based-terraform-module.md) | Lambda with event triggers | SQS, DynamoDB, Kinesis, EventBridge |

## Design Principles

All Terraform modules follow these principles:

### 1. Security First

- **Least Privilege IAM**: Minimal required permissions
- **Encryption**: CloudWatch logs encryption support
- **Scanning**: tfsec, Checkov, Trivy integration
- **Dead Letter Queues**: Error handling for reliability

### 2. Flexibility

- **Optional Components**: Create or use existing resources
- **Configurable**: Extensive variable support
- **Multi-Source**: Support for multiple event sources

### 3. Production Ready

- **Tagging**: Comprehensive resource tagging
- **Outputs**: All important ARNs and URLs exposed
- **Examples**: Complete working examples included

## Module Structure

All modules follow a consistent structure:

```text
module/
├── main.tf           # Core resources
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── versions.tf       # Provider requirements
├── examples/         # Complete usage examples
│   └── basic/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── README.md         # Module documentation
├── CHANGELOG.md      # Version history
├── CONTRIBUTING.md   # Contribution guide
└── SECURITY.md       # Security policies
```

## Common Variables

All modules accept these common variables:

| Variable | Type | Description |
|----------|------|-------------|
| `function_name` | string | Name of the Lambda function |
| `environment_variables` | map(string) | Lambda environment variables |
| `log_retention_days` | number | CloudWatch log retention |
| `tags` | map(string) | Resource tags |

## Security Scanning

Modules are scanned with:

- **tfsec**: Terraform static analysis
- **Checkov**: Infrastructure as code scanning
- **Trivy**: Vulnerability and misconfiguration scanning
- **Gitleaks**: Secret detection in commits

## Usage Pattern

```hcl
module "my_lambda" {
  source = "github.com/riyanimam/<module-name>//opentofu"

  function_name    = "my-function"
  source_code_path = "lambda.zip"

  # Event source configuration
  event_source_type   = "sqs"
  create_event_source = true

  # Customization
  environment_variables = {
    LOG_LEVEL = "INFO"
  }

  tags = {
    Environment = "production"
    Project     = "my-project"
  }
}
```

## Best Practices

### State Management

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "lambda/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### Version Pinning

```hcl
module "my_lambda" {
  source = "github.com/riyanimam/module-name?ref=v1.0.0"
  # ...
}
```

### Workspace Separation

```bash
# Create workspaces for environments
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Apply to specific workspace
terraform workspace select prod
terraform apply
```
