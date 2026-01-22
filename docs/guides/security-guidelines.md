# Security Guidelines

Security best practices for AWS, infrastructure as code, and application development
across all projects.

## Security Principles

### Defense in Depth

Apply multiple layers of security:

```mermaid
flowchart TB
    A[Network Security] --> B[IAM & Access Control]
    B --> C[Application Security]
    C --> D[Data Encryption]
    D --> E[Monitoring & Logging]
```

### Least Privilege

Only grant the minimum permissions required:

=== "Bad ❌"

    ```json
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
    ```

=== "Good ✅"

    ```json
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
    ```

## AWS Security

### IAM Best Practices

| Practice | Description |
|----------|-------------|
| **Use IAM Roles** | Never use long-lived access keys |
| **MFA Required** | Enable MFA for all users |
| **Rotate Credentials** | Regularly rotate access keys |
| **Use Conditions** | Add conditions to policies |
| **Separate Accounts** | Use AWS Organizations |

### IAM Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Access",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::${bucket_name}/*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        },
        "Bool": {
          "aws:SecureTransport": "true"
        }
      }
    }
  ]
}
```

### Lambda Security

```hcl
resource "aws_lambda_function" "secure_lambda" {
  function_name = "secure-function"
  role          = aws_iam_role.lambda_role.arn
  runtime       = "python3.12"
  handler       = "main.handler"

  # Enable X-Ray tracing
  tracing_config {
    mode = "Active"
  }

  # Use VPC for network isolation
  vpc_config {
    subnet_ids         = var.private_subnet_ids
    security_group_ids = [aws_security_group.lambda_sg.id]
  }

  # Encrypt environment variables
  kms_key_arn = aws_kms_key.lambda_key.arn

  environment {
    variables = {
      LOG_LEVEL = "INFO"
    }
  }

  # Reserved concurrency for DDoS protection
  reserved_concurrent_executions = 100

  # Dead letter queue for failed invocations
  dead_letter_config {
    target_arn = aws_sqs_queue.dlq.arn
  }
}
```

### S3 Security

```hcl
resource "aws_s3_bucket" "secure_bucket" {
  bucket = "my-secure-bucket"
}

# Block public access
resource "aws_s3_bucket_public_access_block" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Enable encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.s3_key.arn
    }
    bucket_key_enabled = true
  }
}

# Enable versioning
resource "aws_s3_bucket_versioning" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable logging
resource "aws_s3_bucket_logging" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  target_bucket = aws_s3_bucket.log_bucket.id
  target_prefix = "s3-access-logs/"
}
```

## Infrastructure as Code Security

### Terraform Security

Run security scans in CI:

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # tfsec - Terraform security scanner
      - name: tfsec
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          additional_args: --minimum-severity HIGH

      # Checkov - Policy as code
      - name: Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: .
          framework: terraform

      # Trivy - Vulnerability scanner
      - name: Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "config"
          scan-ref: "."
```

### Common Checkov Policies

| Check ID | Description | Severity |
|----------|-------------|----------|
| CKV_AWS_19 | EBS encryption enabled | HIGH |
| CKV_AWS_24 | Security group no 0.0.0.0/0 | HIGH |
| CKV_AWS_50 | Lambda X-Ray enabled | MEDIUM |
| CKV_AWS_116 | Lambda DLQ configured | MEDIUM |
| CKV_AWS_117 | Lambda in VPC | MEDIUM |
| CKV_AWS_173 | Lambda env vars encrypted | HIGH |

### Fixing Common Issues

=== "CKV_AWS_19: EBS Encryption"

    ```hcl
    resource "aws_ebs_volume" "example" {
      availability_zone = "us-east-1a"
      size              = 40
      encrypted         = true  # Add this
      kms_key_id        = aws_kms_key.ebs_key.arn
    }
    ```

=== "CKV_AWS_24: Security Group"

    ```hcl
    resource "aws_security_group_rule" "example" {
      type              = "ingress"
      from_port         = 443
      to_port           = 443
      protocol          = "tcp"
      # Don't use: cidr_blocks = ["0.0.0.0/0"]
      cidr_blocks       = ["10.0.0.0/8"]  # Restrict to VPC
      security_group_id = aws_security_group.example.id
    }
    ```

## Secret Management

### Never Hardcode Secrets

!!! danger "Never Do This"
    ```python
    # WRONG
    API_KEY = "sk-1234567890abcdef"
    DATABASE_URL = "postgres://user:password@host:5432/db"
    ```

### Use Environment Variables

```python
import os

# Correct approach
API_KEY = os.environ.get("API_KEY")
DATABASE_URL = os.environ.get("DATABASE_URL")

if not API_KEY:
    raise ValueError("API_KEY environment variable not set")
```

### AWS Secrets Manager

```python
import boto3
import json

def get_secret(secret_name: str, region: str = "us-east-1") -> dict:
    """Retrieve secret from AWS Secrets Manager."""
    client = boto3.client("secretsmanager", region_name=region)

    response = client.get_secret_value(SecretId=secret_name)

    if "SecretString" in response:
        return json.loads(response["SecretString"])
    else:
        raise ValueError("Binary secrets not supported")

# Usage
secrets = get_secret("my-app/production")
api_key = secrets["API_KEY"]
```

### Terraform with Secrets Manager

```hcl
data "aws_secretsmanager_secret_version" "db_credentials" {
  secret_id = "my-app/database"
}

locals {
  db_creds = jsondecode(data.aws_secretsmanager_secret_version.db_credentials.secret_string)
}

resource "aws_db_instance" "database" {
  engine               = "postgres"
  engine_version       = "15"
  instance_class       = "db.t3.micro"
  allocated_storage    = 20
  username             = local.db_creds["username"]
  password             = local.db_creds["password"]
  db_name              = local.db_creds["database"]
  skip_final_snapshot  = true
}
```

## Application Security

### Input Validation

```python
from pydantic import BaseModel, validator, constr

class UserInput(BaseModel):
    username: constr(min_length=3, max_length=50, pattern=r"^[a-zA-Z0-9_]+$")
    email: str
    age: int

    @validator("email")
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("Invalid email format")
        return v.lower()

    @validator("age")
    def validate_age(cls, v):
        if not 0 <= v <= 150:
            raise ValueError("Age must be between 0 and 150")
        return v
```

### SQL Injection Prevention

=== "Bad ❌"

    ```python
    # VULNERABLE TO SQL INJECTION
    query = f"SELECT * FROM users WHERE id = {user_id}"
    cursor.execute(query)
    ```

=== "Good ✅"

    ```python
    # Use parameterized queries
    query = "SELECT * FROM users WHERE id = %s"
    cursor.execute(query, (user_id,))
    ```

### Logging Security

```python
import logging

logger = logging.getLogger(__name__)

def process_user(user_id: str, api_key: str):
    # Never log sensitive data
    logger.info(f"Processing user: {user_id}")

    # NEVER DO THIS
    # logger.info(f"Using API key: {api_key}")

    # If needed, mask sensitive data
    masked_key = f"{api_key[:4]}...{api_key[-4:]}" if len(api_key) > 8 else "***"
    logger.debug(f"Using key ending in: {masked_key}")
```

## Dependency Security

### Dependabot Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      development-dependencies:
        patterns:
          - "@types/*"
          - "*eslint*"
          - "*prettier*"

  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "terraform"
    directory: "/terraform"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Lock Files

Always commit lock files:

| Ecosystem | Lock File |
|-----------|-----------|
| npm | `package-lock.json` |
| pnpm | `pnpm-lock.yaml` |
| pip | `requirements.txt` (pinned) |
| Go | `go.sum` |

### Audit Commands

```bash
# Node.js
npm audit
pnpm audit

# Python
pip-audit

# Go
go list -json -m all | nancy sleuth
```

## Git Security

### Pre-commit Secret Scanning

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: detect-private-key
      - id: detect-aws-credentials
```

### .gitignore Security

```gitignore
# Environment files
.env
.env.local
.env.*.local
*.env

# AWS credentials
.aws/credentials
*.pem
*.key

# Terraform state (contains secrets)
*.tfstate
*.tfstate.*
.terraform/

# IDE settings that may contain paths
.idea/
.vscode/settings.json
```

### Signed Commits

```bash
# Generate GPG key
gpg --full-generate-key

# List keys
gpg --list-secret-keys --keyid-format=long

# Configure Git
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# Sign commits
git commit -S -m "Signed commit"
```

## Monitoring & Incident Response

### CloudWatch Alarms

```hcl
resource "aws_cloudwatch_metric_alarm" "high_error_rate" {
  alarm_name          = "lambda-high-error-rate"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = 300
  statistic           = "Sum"
  threshold           = 10
  alarm_description   = "Lambda function error rate is too high"

  dimensions = {
    FunctionName = aws_lambda_function.main.function_name
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}
```

### Incident Response Checklist

1. **Detect** - Alert triggered or issue reported
2. **Assess** - Determine scope and severity
3. **Contain** - Stop the bleeding
4. **Eradicate** - Remove the threat
5. **Recover** - Restore normal operations
6. **Review** - Post-incident analysis

## Security Checklist

### Development

- [ ] No hardcoded secrets in code
- [ ] Input validation on all user input
- [ ] Parameterized queries for databases
- [ ] Secure password hashing (bcrypt, argon2)
- [ ] HTTPS for all communications

### Infrastructure

- [ ] IAM follows least privilege
- [ ] Encryption at rest enabled
- [ ] Encryption in transit enabled
- [ ] VPC for network isolation
- [ ] Security groups properly configured

### CI/CD

- [ ] Secret scanning enabled
- [ ] Dependency scanning enabled
- [ ] SAST/DAST scanning enabled
- [ ] Signed commits required
- [ ] Branch protection enabled

### Monitoring

- [ ] CloudWatch logging enabled
- [ ] Alarms configured for anomalies
- [ ] Audit trails enabled (CloudTrail)
- [ ] Regular security reviews scheduled

## Resources

- [AWS Security Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [OWASP Top 10](https://owasp.org/Top10/)
- [CIS AWS Benchmarks](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Terraform Security Best Practices](https://developer.hashicorp.com/terraform/cloud-docs/recommended-practices/security)
- [GitHub Security Best Practices](https://docs.github.com/en/code-security)
