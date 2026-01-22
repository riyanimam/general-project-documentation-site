# Lambda Debugging

Debugging AWS Lambda functions, CloudWatch Logs, and event handling.

## CloudWatch Logs

### Viewing Logs

=== "AWS CLI"

    ```bash
    # Tail logs
    aws logs tail /aws/lambda/<function-name> --follow

    # Get recent logs
    aws logs tail /aws/lambda/<function-name> --since 1h

    # Filter logs
    aws logs filter-log-events \
      --log-group-name /aws/lambda/<function-name> \
      --filter-pattern "ERROR"
    ```

=== "Console"

    1. Go to AWS Lambda Console
    2. Select your function
    3. Click "Monitor" tab
    4. Click "View logs in CloudWatch"

### Log Insights Queries

```sql
-- Find errors
fields @timestamp, @message
| filter @message like /ERROR|Exception|error/
| sort @timestamp desc
| limit 100

-- Cold starts
fields @timestamp, @message, @duration
| filter @message like /INIT_START/
| stats count() as coldStarts by bin(1h)

-- Duration analysis
fields @timestamp, @duration
| stats avg(@duration) as avgDuration,
        max(@duration) as maxDuration,
        min(@duration) as minDuration
  by bin(1h)

-- Memory usage
fields @timestamp, @maxMemoryUsed, @memorySize
| filter @maxMemoryUsed > 0
| stats avg(@maxMemoryUsed) as avgMemory by bin(1h)
```

### Structured Logging

Implement structured logging for better debugging:

=== "Python"

    ```python
    import json
    import logging

    logger = logging.getLogger()
    logger.setLevel(logging.INFO)

    def lambda_handler(event, context):
        # Log with structure
        logger.info(json.dumps({
            "message": "Processing request",
            "request_id": context.aws_request_id,
            "event_source": event.get("source"),
            "record_count": len(event.get("Records", []))
        }))

        try:
            result = process(event)
            logger.info(json.dumps({
                "message": "Success",
                "result": result
            }))
            return result
        except Exception as e:
            logger.error(json.dumps({
                "message": "Error processing request",
                "error": str(e),
                "error_type": type(e).__name__
            }))
            raise
    ```

=== "TypeScript"

    ```typescript
    interface LogEntry {
      message: string;
      requestId: string;
      [key: string]: unknown;
    }

    function log(entry: LogEntry): void {
      console.log(JSON.stringify(entry));
    }

    export const handler = async (event: any, context: any) => {
      log({
        message: "Processing request",
        requestId: context.awsRequestId,
        recordCount: event.Records?.length ?? 0,
      });

      try {
        const result = await process(event);
        log({ message: "Success", requestId: context.awsRequestId, result });
        return result;
      } catch (error) {
        log({
          message: "Error",
          requestId: context.awsRequestId,
          error: String(error),
        });
        throw error;
      }
    };
    ```

=== "Go"

    ```go
    package main

    import (
        "context"
        "encoding/json"
        "log"

        "github.com/aws/aws-lambda-go/lambdacontext"
    )

    type LogEntry struct {
        Message   string `json:"message"`
        RequestID string `json:"requestId"`
        Error     string `json:"error,omitempty"`
    }

    func logEntry(entry LogEntry) {
        data, _ := json.Marshal(entry)
        log.Println(string(data))
    }

    func handler(ctx context.Context, event interface{}) error {
        lc, _ := lambdacontext.FromContext(ctx)

        logEntry(LogEntry{
            Message:   "Processing request",
            RequestID: lc.AwsRequestID,
        })

        // Process...

        return nil
    }
    ```

## Common Lambda Issues

### Timeout Errors

**Symptom:**

```
Task timed out after X seconds
```

**Causes:**

- Operation takes longer than timeout
- Network issues
- Deadlock or infinite loop

**Solutions:**

1. **Increase timeout** (if operation legitimately takes time):

   ```hcl
   resource "aws_lambda_function" "main" {
     timeout = 300  # 5 minutes max for sync, 15 for async
   }
   ```

2. **Optimize code**:

   ```python
   # Use connection pooling
   import boto3
   from botocore.config import Config

   config = Config(
       max_pool_connections=25,
       retries={'max_attempts': 3}
   )
   client = boto3.client('s3', config=config)
   ```

3. **Check for network issues** in VPC:
   - Ensure NAT Gateway for internet access
   - Check security group rules
   - Verify VPC endpoints for AWS services

### Memory Errors

**Symptom:**

```
Runtime exited with error: signal: killed
```

**Solution:**

```hcl
resource "aws_lambda_function" "main" {
  memory_size = 512  # Increase memory
}
```

Monitor memory usage:

```sql
-- CloudWatch Logs Insights
fields @timestamp, @maxMemoryUsed, @memorySize
| stats max(@maxMemoryUsed) / @memorySize * 100 as memoryUtilization
```

### Cold Starts

**Symptom:**

```
Slow first invocation, INIT_START in logs
```

**Solutions:**

1. **Provisioned Concurrency**:

   ```hcl
   resource "aws_lambda_provisioned_concurrency_config" "main" {
     function_name                     = aws_lambda_function.main.function_name
     provisioned_concurrent_executions = 5
     qualifier                         = aws_lambda_alias.live.name
   }
   ```

2. **Optimize initialization**:

   ```python
   # Move heavy imports/setup outside handler
   import boto3

   # This runs once per container
   s3_client = boto3.client('s3')
   dynamodb = boto3.resource('dynamodb')
   table = dynamodb.Table(os.environ['TABLE_NAME'])

   def handler(event, context):
       # Use pre-initialized clients
       pass
   ```

3. **Smaller package size**:
   - Use Lambda layers
   - Remove unnecessary dependencies
   - Use lighter alternatives (e.g., orjson vs json)

### Import Errors

**Symptom:**

```
Unable to import module 'handler': No module named 'package'
```

**Solutions:**

1. **Check package is in deployment**:

   ```bash
   # Verify zip contents
   unzip -l lambda.zip | grep package_name
   ```

2. **Correct directory structure**:

   ```
   lambda.zip
   ├── handler.py
   └── package/
       └── __init__.py
   ```

3. **Use Lambda Layers** for dependencies:

   ```hcl
   resource "aws_lambda_function" "main" {
     layers = [aws_lambda_layer_version.dependencies.arn]
   }
   ```

### Permission Errors

**Symptom:**

```
AccessDeniedException: User is not authorized to perform: X on resource: Y
```

**Solution:**

Check IAM role permissions:

```bash
# Get role policies
aws iam list-attached-role-policies --role-name <role-name>
aws iam list-role-policies --role-name <role-name>

# Get specific policy
aws iam get-role-policy --role-name <role-name> --policy-name <policy-name>
```

Add required permissions:

```hcl
resource "aws_iam_role_policy" "lambda_policy" {
  name = "lambda_policy"
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["s3:GetObject", "s3:PutObject"]
        Resource = "${aws_s3_bucket.bucket.arn}/*"
      }
    ]
  })
}
```

## Event Source Debugging

### SQS Issues

**Messages not being processed:**

1. Check event source mapping is enabled:

   ```bash
   aws lambda list-event-source-mappings --function-name <name>
   ```

2. Check SQS queue attributes:

   ```bash
   aws sqs get-queue-attributes \
     --queue-url <url> \
     --attribute-names All
   ```

3. Verify visibility timeout >= 6x Lambda timeout

**Messages going to DLQ:**

Check for processing errors in CloudWatch Logs:

```sql
fields @timestamp, @message
| filter @message like /ERROR|Exception/
| sort @timestamp desc
```

### DynamoDB Streams

**Stream not triggering Lambda:**

1. Check stream is enabled:

   ```bash
   aws dynamodb describe-table --table-name <table>
   ```

2. Verify event source mapping:

   ```bash
   aws lambda list-event-source-mappings --function-name <name>
   ```

3. Check shard iterator:

   ```bash
   aws dynamodbstreams get-shard-iterator \
     --stream-arn <stream-arn> \
     --shard-id <shard-id> \
     --shard-iterator-type LATEST
   ```

### API Gateway

**502 Bad Gateway:**

1. Check Lambda response format:

   ```python
   def handler(event, context):
       return {
           'statusCode': 200,
           'headers': {'Content-Type': 'application/json'},
           'body': json.dumps({'message': 'Success'})
       }
   ```

2. Check Lambda timeout (API Gateway has 29s limit)

3. Enable CloudWatch Logs for API Gateway:

   ```bash
   aws apigateway update-stage \
     --rest-api-id <api-id> \
     --stage-name prod \
     --patch-operations op=replace,path=/accessLogSettings/destinationArn,value=<log-group-arn>
   ```

## X-Ray Tracing

### Enable X-Ray

```hcl
resource "aws_lambda_function" "main" {
  tracing_config {
    mode = "Active"
  }
}
```

### Add Instrumentation

=== "Python"

    ```python
    from aws_xray_sdk.core import xray_recorder
    from aws_xray_sdk.core import patch_all

    patch_all()

    @xray_recorder.capture('process_data')
    def process_data(data):
        # Your code here
        pass

    def handler(event, context):
        with xray_recorder.in_segment('handler') as segment:
            segment.put_annotation('key', 'value')
            segment.put_metadata('data', event)
            return process_data(event)
    ```

=== "TypeScript"

    ```typescript
    import * as AWSXRay from "aws-xray-sdk-core";
    import * as AWS from "aws-sdk";

    const aws = AWSXRay.captureAWS(AWS);

    export const handler = async (event: any) => {
      const s3 = new aws.S3();
      // S3 calls are now traced
    };
    ```

### Analyze Traces

1. Go to AWS X-Ray Console
2. Click "Traces"
3. Filter by time range and service
4. Click on a trace to see the timeline

## Local Testing

### Using SAM CLI

```bash
# Install SAM CLI
brew install aws-sam-cli

# Invoke locally
sam local invoke MyFunction -e event.json

# Start local API
sam local start-api

# Generate sample events
sam local generate-event sqs receive-message > event.json
```

### Using Docker

```bash
# Run Lambda container
docker run --rm \
  -v "$PWD":/var/task \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  public.ecr.aws/lambda/python:3.12 \
  handler.lambda_handler '{"key": "value"}'
```

### Unit Testing

=== "Python"

    ```python
    import pytest
    from unittest.mock import patch, MagicMock
    from handler import lambda_handler

    @pytest.fixture
    def mock_context():
        context = MagicMock()
        context.aws_request_id = "test-request-id"
        return context

    def test_handler_success(mock_context):
        event = {"Records": [{"body": "test"}]}

        with patch('handler.process_record') as mock_process:
            mock_process.return_value = {"status": "success"}
            result = lambda_handler(event, mock_context)

        assert result["statusCode"] == 200
    ```

=== "TypeScript"

    ```typescript
    import { describe, it, expect, vi } from "vitest";
    import { handler } from "./handler.js";

    describe("handler", () => {
      it("should process event successfully", async () => {
        const event = { Records: [{ body: "test" }] };
        const context = { awsRequestId: "test-id" } as any;

        const result = await handler(event, context);

        expect(result.statusCode).toBe(200);
      });
    });
    ```

## Debugging Checklist

- [ ] Check CloudWatch Logs for errors
- [ ] Verify IAM permissions
- [ ] Confirm event source mapping is enabled
- [ ] Check timeout and memory settings
- [ ] Verify network configuration (VPC, security groups)
- [ ] Test with sample event locally
- [ ] Check X-Ray traces for bottlenecks
- [ ] Verify environment variables are set
- [ ] Confirm deployment package has all dependencies
