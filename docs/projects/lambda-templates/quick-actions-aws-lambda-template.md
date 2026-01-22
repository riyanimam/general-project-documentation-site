# Quick Actions AWS Lambda Template

A production-ready AWS Lambda starter kit featuring Python 3.12, Ruff linting, Terraform
infrastructure, and automated CI/CD pipelines. Provides 18+ pre-built AWS service actions.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/quick-actions-aws-lambda-template){ .md-button }

## Features

- **AWS Lambda Function**: Python 3.12 Lambda with SQS trigger
- **Infrastructure as Code**: All AWS resources managed via Terraform
- **CI/CD**: GitHub Actions pipeline for linting, formatting, and Terraform
- **Code Quality**: Python linting and formatting with Ruff
- **Best Practices**: Environment variables, tagging, log retention, least-privilege IAM

## Supported Actions

The Lambda function provides utility actions across multiple AWS services:

| Service | Actions |
|---------|---------|
| **Lambda** | `delete_lambda_function`, `list_lambda_functions` |
| **SQS** | `redrive_sqs_dlq`, `send_sqs_message` |
| **ECR** | `get_ecr_login_and_repo_uri`, `list_ecr_repositories` |
| **ECS Fargate** | `list_ecs_clusters`, `run_fargate_task` |
| **DynamoDB** | `put_dynamodb_item`, `query_dynamodb` |
| **CloudWatch** | `put_cloudwatch_metric`, `get_cloudwatch_metric_statistics` |
| **SNS** | `publish_sns_message`, `list_sns_topics` |
| **S3** | `upload_file_to_s3`, `list_s3_objects` |
| **Glue** | `start_glue_job`, `get_glue_job_run` |

## Project Structure

```text
.
├── python/
│   ├── src/
│   │   └── base_lambda.py      # Lambda function source code
│   ├── test/
│   │   └── test_base_lambda.py # Unit tests
│   └── integration/
│       └── integration_base_lambda.py  # Integration tests
├── terraform/
│   ├── lambda.tf               # Lambda, SQS, CloudWatch
│   ├── iam_lambda.tf           # IAM roles and policies
│   └── providers.tf            # Terraform providers
├── scripts/
│   └── build.sh                # Build deployment package
├── .github/
│   └── workflows/
│       └── ci.yml              # CI/CD pipeline
└── README.md
```

## Quick Start

### Prerequisites

- Python 3.12+
- Terraform
- AWS CLI
- Docker (optional, for packaging)

### Setup

```bash
# Clone the repository
git clone https://github.com/riyanimam/quick-actions-aws-lambda-template.git
cd quick-actions-aws-lambda-template

# Install Python dependencies
pip install -r requirements.txt

# Build Lambda deployment package
mkdir -p build
cd python/src
zip -r ../../build/example_lambda.zip .
cd ../../

# Initialize and apply Terraform
cd terraform
terraform init
terraform apply
```

## Usage

The Lambda function expects an event with a `body` containing the action:

### Delete Lambda Function

```json
{
  "body": "{\"event\": \"delete_lambda_function\", \"function_name\": \"target_lambda\"}"
}
```

### Redrive SQS Dead Letter Queue

```json
{
  "body": "{\"event\": \"redrive_sqs_dlq\", \"source_queue_url\": \"src\", \"dlq_url\": \"dlq\"}"
}
```

### Run Fargate Task

```json
{
  "body": "{\"event\": \"run_fargate_task\", \"cluster\": \"my-cluster\", \"task_definition\": \"my-task-def\", \"subnets\": [\"subnet-xxxxxx\"]}"
}
```

## Lambda Handler

The handler uses an event-driven dispatch pattern:

```python
def lambda_handler(payload: JSONType, context: LambdaContext):
    event = json.loads(payload["body"])

    if event == "delete_lambda_function":
        delete_lambda_function("function_name_here")
    elif event == "redrive_sqs_dlq":
        redrive_sqs_dlq("source_queue_url_here", "dlq_url_here")
    elif event == "list_lambda_functions":
        return list_lambda_functions()
    # ... more actions
    else:
        return "Invalid or no event received"
```

### AWS Client Configuration

All AWS clients use a standardized configuration for reliability:

```python
from botocore.config import Config

aws_client_config = Config(
    connect_timeout=10,
    read_timeout=10,
    retries={"max_attempts": 4, "mode": "standard"}
)
```

## Terraform Infrastructure

### Resources Created

- **Lambda Function** (`example_lambda`)
- **IAM Role** for Lambda execution
- **CloudWatch Log Group** for Lambda logs
- **SQS Queue** as event source
- **Event Source Mapping** between SQS and Lambda

### Lambda Configuration

```hcl
resource "aws_lambda_function" "example_lambda" {
  function_name = "example_lambda"
  filename      = "build/example_lambda.zip"
  handler       = "base_lambda.lambda_handler"
  runtime       = "python3.12"
  role          = aws_iam_role.lambda_exec.arn

  environment {
    variables = {
      ENV = "production"
    }
  }

  tags = {
    Project     = "QuickActions"
    Environment = "Production"
  }
}
```

## CI/CD Pipeline

GitHub Actions workflow includes:

### Code Quality Stage

- Lint Python with Ruff
- Check Python formatting with Ruff Format
- Check Terraform formatting
- Lint TOML files
- Lint YAML files
- Check Markdown formatting

### Terraform Stage

- Terraform init, plan, and apply (on main branch)

## Testing

### Unit Tests

```bash
# Run unit tests
pytest python/test/ -v

# Run with coverage
pytest python/test/ -v --cov=python/src
```

### Integration Tests

```bash
# Run integration tests (requires AWS credentials)
pytest python/integration/ -v
```

### Example Test

```python
@mock.patch("src.base_lambda.boto3.client")
def test_delete_lambda_function_calls_boto3(mock_boto_client):
    mock_lambda = mock.Mock()
    mock_boto_client.return_value = mock_lambda
    mock_lambda.delete_function.return_value = {
        "ResponseMetadata": {"HTTPStatusCode": 204}
    }

    resp = base_lambda.delete_lambda_function("my-func")
    mock_boto_client.assert_called_once_with(
        "lambda", config=base_lambda.aws_client_config
    )
    mock_lambda.delete_function.assert_called_once_with(FunctionName="my-func")
```

## Gotchas & Tips

!!! warning "Event Body Parsing"
    The handler expects `payload["body"]` to be a JSON string. Ensure your
    event source (SQS, API Gateway) sends the body in this format.

!!! tip "Error Handling"
    Each action function has its own try/except block. Errors are logged
    but don't crash the handler, allowing batch processing to continue.

!!! note "IAM Permissions"
    The default IAM policy includes broad permissions for demonstration.
    **Always** restrict to least-privilege for production use.

## Troubleshooting

### Lambda Invocation Errors

Check CloudWatch Logs for detailed error messages:

```bash
aws logs tail /aws/lambda/example_lambda --follow
```

### Missing Permissions

If you see `AccessDeniedException`:

1. Review the IAM policy in `terraform/iam_lambda.tf`
2. Add the required action to the policy
3. Redeploy with `terraform apply`

### SQS Message Not Processing

1. Check the SQS queue for messages
2. Verify the event source mapping is enabled
3. Check Lambda concurrent execution limits

## Customization

### Add New Actions

1. Add a new function in `python/src/base_lambda.py`
2. Add the event handler in `lambda_handler`
3. Add unit tests in `python/test/test_base_lambda.py`
4. Update IAM policy if new permissions needed

### Modify Terraform

1. Update the relevant `.tf` file
2. Run `terraform plan` to preview changes
3. Apply with `terraform apply`

## Enhancement Ideas

- [ ] Add API Gateway integration
- [ ] Implement request validation with Pydantic
- [ ] Add Step Functions orchestration
- [ ] Support for Lambda Layers
- [ ] Add CloudWatch Alarms for errors
- [ ] Implement cross-account actions

## Resources

- [AWS Lambda Python](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html)
- [Ruff Documentation](https://docs.astral.sh/ruff/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [AWS Lambda Powertools](https://docs.powertools.aws.dev/lambda/python/latest/)
