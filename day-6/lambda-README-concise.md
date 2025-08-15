# AWS Lambda - README

## 📘 What is AWS Lambda?

**AWS Lambda** is a **serverless compute service** that runs your code in response to events without managing servers. You pay only for the compute time you consume.

**Key Triggers:**

- HTTP requests (API Gateway)
- File uploads (S3)
- Database changes (DynamoDB)
- Queue messages (SQS)
- Scheduled events (CloudWatch)

---

## 🚀 Why Use Lambda?

- ✅ **No Server Management** - Focus on code, not infrastructure
- 🔄 **Auto-scaling** - From zero to thousands of executions
- 💰 **Pay-per-use** - Only pay for execution time
- ⚡ **Fast deployment** - Deploy in seconds
- 🔐 **Built-in security** - IAM and VPC integration

---

## 🛠️ Quick Start

### 1. Basic Function (Python)

```python
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

### 2. Create via AWS CLI

```bash
# Package your code
zip function.zip lambda_function.py

# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### 3. Using SAM Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Runtime: python3.9
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

---

## 💡 Common Use Cases

### 1. REST API

```python
def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']

    if method == 'GET' and path == '/users':
        return {
            'statusCode': 200,
            'body': json.dumps([{'id': 1, 'name': 'John'}])
        }

    return {'statusCode': 404, 'body': 'Not Found'}
```

### 2. File Processing

```python
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')

    # Get file info from S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    # Process file
    obj = s3.get_object(Bucket=bucket, Key=key)
    content = obj['Body'].read()

    # Do something with content
    processed = content.upper()

    # Save result
    s3.put_object(
        Bucket=bucket,
        Key=f"processed/{key}",
        Body=processed
    )

    return {'statusCode': 200}
```

### 3. Scheduled Task

```python
import boto3
from datetime import datetime, timedelta

def lambda_handler(event, context):
    s3 = boto3.client('s3')

    # Delete files older than 7 days
    cutoff = datetime.now() - timedelta(days=7)

    response = s3.list_objects_v2(Bucket='my-bucket', Prefix='temp/')

    for obj in response.get('Contents', []):
        if obj['LastModified'].replace(tzinfo=None) < cutoff:
            s3.delete_object(Bucket='my-bucket', Key=obj['Key'])

    return {'statusCode': 200}
```

---

## 🔧 Configuration Files

### buildspec.yml (for CodeBuild)

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      python: 3.9
  build:
    commands:
      - pip install -r requirements.txt
      - python -m pytest
artifacts:
  files:
    - "**/*"
```

### appspec.yml (for CodeDeploy)

```yaml
version: 0.0
Resources:
  - MyLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "my-function"
        Alias: "live"
        CurrentVersion: "1"
        TargetVersion: "2"
```

---

## 🔐 IAM Role Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

---

## 📊 Best Practices

### Performance

- Initialize connections outside the handler
- Use environment variables for configuration
- Right-size memory allocation
- Use provisioned concurrency for consistent performance

### Security

- Follow least privilege principle for IAM roles
- Store secrets in AWS Secrets Manager
- Enable VPC if accessing private resources
- Use resource-based policies for cross-account access

### Monitoring

```python
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(f"Processing event: {event}")
    # Your code here
    return {'statusCode': 200}
```

---

## 💰 Pricing

- **Free Tier**: 1M requests + 400,000 GB-seconds/month
- **Requests**: $0.20 per 1M requests
- **Duration**: $0.0000166667 per GB-second

**Example**: 512MB function, 100ms duration, 1M invocations/month = ~$1.03

---

## 🆘 Common Issues

| Issue           | Solution                                      |
| --------------- | --------------------------------------------- |
| **Timeout**     | Increase timeout setting, optimize code       |
| **Memory**      | Monitor usage, increase allocation if needed  |
| **Cold starts** | Use provisioned concurrency, optimize imports |
| **Permissions** | Check IAM roles and resource policies         |

---

## 🧪 Testing

### Local Test

```python
# test_lambda.py
from lambda_function import lambda_handler

def test_handler():
    event = {'test': 'data'}
    context = type('Context', (), {'aws_request_id': 'test-id'})()

    response = lambda_handler(event, context)
    assert response['statusCode'] == 200

if __name__ == '__main__':
    test_handler()
    print("Test passed!")
```

### Using Moto for AWS Mocking

```python
import boto3
from moto import mock_s3
from lambda_function import lambda_handler

@mock_s3
def test_s3_function():
    # Create mock S3
    s3 = boto3.client('s3', region_name='us-east-1')
    s3.create_bucket(Bucket='test-bucket')

    # Test your function
    response = lambda_handler(test_event, None)
    assert response['statusCode'] == 200
```

---

## 📖 Key Resources

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [Serverless Framework](https://www.serverless.com/)
- [Lambda Powertools](https://awslabs.github.io/aws-lambda-powertools-python/)

---

**AWS Lambda: Run code without thinking about servers. Scale automatically. Pay only for what you use.**
