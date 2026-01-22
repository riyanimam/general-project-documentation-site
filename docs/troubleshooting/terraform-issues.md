# Terraform Issues

Terraform state, provider, and deployment troubleshooting.

## State Issues

### State Lock Stuck

**Symptom:**
```
Error acquiring the state lock
Lock Info:
  ID:        xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
  Path:      s3://bucket/terraform.tfstate
```

**Solution:**
```bash
# Force unlock (use with caution!)
terraform force-unlock <LOCK_ID>

# Verify no other operations are running first
aws dynamodb get-item \
  --table-name terraform-locks \
  --key '{"LockID": {"S": "bucket/terraform.tfstate"}}'
```

### State Drift

**Symptom:**
```
Note: Objects have changed outside of Terraform
```

**Solution:**
```bash
# Refresh state
terraform refresh

# Or apply to reconcile
terraform apply -refresh-only
```

### Resource Already Exists

**Symptom:**
```
Error: A resource with the ID "xxx" already exists
```

**Solution:**
```bash
# Import existing resource
terraform import aws_lambda_function.main my-function

# Example imports
terraform import aws_iam_role.lambda_role lambda-role-name
terraform import aws_s3_bucket.bucket bucket-name
terraform import 'aws_sqs_queue.queue' https://sqs.us-east-1.amazonaws.com/123456789012/my-queue
```

### State File Corruption

**Symptom:**
```
Error loading state: state file could not be parsed
```

**Solution:**
```bash
# Pull state from remote
terraform state pull > state.json

# Check for valid JSON
jq '.' state.json

# If backup exists
cp terraform.tfstate.backup terraform.tfstate
```

## Provider Issues

### Provider Version Conflicts

**Symptom:**
```
Error: Failed to query available provider packages
```

**Solution:**
```hcl
# Pin provider versions
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

Then:
```bash
# Update lock file
terraform init -upgrade
```

### Authentication Errors

**Symptom:**
```
Error: No valid credential sources found
```

**Solution:**
```bash
# Configure credentials
aws configure

# Or use environment variables
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_DEFAULT_REGION="us-east-1"

# Or use AWS SSO
aws sso login --profile your-profile
export AWS_PROFILE=your-profile

# Verify
aws sts get-caller-identity
```

### Missing Provider

**Symptom:**
```
Error: Could not satisfy plugin requirements
```

**Solution:**
```bash
# Clear cache and reinitialize
rm -rf .terraform
rm .terraform.lock.hcl
terraform init
```

## Validation Errors

### Invalid Resource Configuration

**Symptom:**
```
Error: Invalid value for variable
```

**Solution:**

1. Check variable types:
   ```hcl
   variable "port" {
     type        = number  # Not string!
     description = "Port number"
   }
   ```

2. Add validation:
   ```hcl
   variable "environment" {
     type = string
     validation {
       condition     = contains(["dev", "staging", "prod"], var.environment)
       error_message = "Environment must be dev, staging, or prod."
     }
   }
   ```

### Cycle Detected

**Symptom:**
```
Error: Cycle: resource A → resource B → resource A
```

**Solution:**

Break the cycle using `depends_on` or restructure:

```hcl
# Before (cycle)
resource "aws_security_group" "a" {
  ingress {
    security_groups = [aws_security_group.b.id]
  }
}

resource "aws_security_group" "b" {
  ingress {
    security_groups = [aws_security_group.a.id]
  }
}

# After (use separate rules)
resource "aws_security_group" "a" {}
resource "aws_security_group" "b" {}

resource "aws_security_group_rule" "a_from_b" {
  security_group_id        = aws_security_group.a.id
  source_security_group_id = aws_security_group.b.id
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
}
```

### Unknown Values

**Symptom:**
```
Error: Invalid count argument - The "count" value depends on resource attributes that cannot be determined until apply
```

**Solution:**

Use `length()` with known values:
```hcl
# Before
count = aws_instance.example.count  # Unknown at plan

# After
variable "instance_count" {
  default = 3
}
count = var.instance_count  # Known at plan
```

## Deployment Issues

### Resource Not Found

**Symptom:**
```
Error: error reading Lambda Function: ResourceNotFoundException
```

**Solution:**

1. Check resource exists:
   ```bash
   aws lambda get-function --function-name my-function
   ```

2. Verify region:
   ```hcl
   provider "aws" {
     region = "us-east-1"  # Correct region?
   }
   ```

3. Check for typos in resource name

### Permission Denied

**Symptom:**
```
Error: error creating IAM Role: AccessDenied
```

**Solution:**

1. Check IAM permissions:
   ```bash
   aws iam simulate-principal-policy \
     --policy-source-arn arn:aws:iam::123456789012:user/terraform \
     --action-names iam:CreateRole
   ```

2. Add required permissions to Terraform IAM user/role

### Rate Limiting

**Symptom:**
```
Error: ThrottlingException: Rate exceeded
```

**Solution:**

Add retry configuration:
```hcl
provider "aws" {
  retry_mode  = "standard"
  max_retries = 10
}
```

Or use a slower parallelism:
```bash
terraform apply -parallelism=5
```

## Module Issues

### Module Not Found

**Symptom:**
```
Error: Module not found
```

**Solution:**
```bash
# Initialize modules
terraform init

# Update modules
terraform get -update

# Check module source
module "example" {
  source = "git::https://github.com/user/repo.git?ref=v1.0.0"
  # Or local path
  source = "./modules/my-module"
}
```

### Module Version Conflict

**Symptom:**
```
Error: Module version requirements conflict
```

**Solution:**
```hcl
module "example" {
  source  = "registry.terraform.io/hashicorp/example/aws"
  version = "~> 3.0"  # Pin version range
}
```

## Backend Issues

### S3 Backend Configuration

**Symptom:**
```
Error: Failed to get existing workspaces
```

**Solution:**

1. Check bucket exists:
   ```bash
   aws s3 ls s3://your-terraform-state-bucket
   ```

2. Check DynamoDB table:
   ```bash
   aws dynamodb describe-table --table-name terraform-locks
   ```

3. Verify backend configuration:
   ```hcl
   terraform {
     backend "s3" {
       bucket         = "your-terraform-state-bucket"
       key            = "path/to/terraform.tfstate"
       region         = "us-east-1"
       dynamodb_table = "terraform-locks"
       encrypt        = true
     }
   }
   ```

### Backend Migration

```bash
# Migrate from local to S3
terraform init -migrate-state

# Reconfigure backend
terraform init -reconfigure
```

## Debugging Commands

### Verbose Logging

```bash
# Enable trace logging
export TF_LOG=TRACE
export TF_LOG_PATH=terraform.log

terraform plan

# View logs
cat terraform.log
```

### State Inspection

```bash
# List resources
terraform state list

# Show resource details
terraform state show aws_lambda_function.main

# Show full state
terraform show

# Output as JSON
terraform show -json > state.json
```

### Plan Analysis

```bash
# Save plan
terraform plan -out=plan.tfplan

# Show plan details
terraform show plan.tfplan

# Show as JSON
terraform show -json plan.tfplan > plan.json

# Check what will be destroyed
terraform plan -destroy
```

## Prevention Best Practices

### Use Version Constraints

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Use Workspaces for Environments

```bash
# Create workspace
terraform workspace new production

# Switch workspace
terraform workspace select production

# List workspaces
terraform workspace list
```

### Use Lifecycle Rules

```hcl
resource "aws_instance" "main" {
  lifecycle {
    prevent_destroy = true  # Prevent accidental deletion
    create_before_destroy = true  # Zero-downtime updates
    ignore_changes = [tags]  # Ignore external changes
  }
}
```

### Run Pre-commit Checks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.88.0
    hooks:
      - id: terraform_fmt
      - id: terraform_validate
      - id: terraform_tflint
      - id: terraform_docs
```

## Debugging Checklist

- [ ] Run `terraform init` to ensure providers are installed
- [ ] Check `terraform validate` for syntax errors
- [ ] Review `terraform plan` output carefully
- [ ] Verify AWS credentials are configured
- [ ] Check region is correct
- [ ] Verify state is not locked
- [ ] Check for resource name collisions
- [ ] Review IAM permissions
- [ ] Enable debug logging if needed
- [ ] Check backend configuration
