# AWS Lambda & Serverless Computing

## 🌟 What is Serverless?

**Serverless** = No server management. Focus on code, AWS handles infrastructure.

**Benefits:** Auto-scaling • Pay-per-execution • Event-driven • Zero maintenance

## 📘 AWS Lambda

**AWS Lambda** runs code without servers. Event triggers → Code executes → Pay only for runtime.

**Triggers:** API Gateway • S3 • DynamoDB • SQS • CloudWatch • IoT  
**Languages:** Python, Node.js, Java, Go, C#, Ruby

## 🛠️ Quick Start

```python
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps(f"Hello {event.get('name', 'World')}")
    }
```

```bash
# Deploy
zip function.zip lambda_function.py
aws lambda create-function --function-name my-app --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler --zip-file fileb://function.zip
```

## 💡 Use Cases

### REST API

```python
def lambda_handler(event, context):
    if event['httpMethod'] == 'GET':
        return {'statusCode': 200, 'body': json.dumps([{'id': 1, 'name': 'Alice'}])}
    return {'statusCode': 404, 'body': 'Not Found'}
```

### File Processing (S3 Trigger)

```python
def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    # Process uploaded file
    return {'statusCode': 200}
```

### Scheduled Tasks

```python
def lambda_handler(event, context):
    # Daily cleanup job
    # Delete old files, send reports, etc.
    return {'statusCode': 200, 'body': 'Cleanup completed'}
```

## 🔐 Best Practices

### IAM Role (Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["logs:*"],
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

### Performance Tips

```python
import boto3
import os

# Initialize outside handler (reuse connections)
s3_client = boto3.client('s3')
TABLE_NAME = os.environ['TABLE_NAME']

def lambda_handler(event, context):
    # Use environment variables and pre-initialized clients
    return {'statusCode': 200}
```

## 💰 Pricing

- **Free Tier**: 1M requests + 400,000 GB-seconds/month
- **Requests**: $0.20 per 1M requests
- **Duration**: $0.0000166667 per GB-second

**Example**: 512MB function, 100ms duration, 1M executions = ~$1/month

## 🆘 Common Issues

| Issue                   | Solution                                           |
| ----------------------- | -------------------------------------------------- |
| **Cold Starts**         | Use Provisioned Concurrency for critical functions |
| **Timeout (15min max)** | Break into smaller functions or use Step Functions |
| **Memory Errors**       | Monitor CloudWatch, increase memory allocation     |
| **Permission Denied**   | Check IAM role policies                            |

## 🎯 When to Use

**✅ Perfect for:** Event-driven apps • APIs • File processing • Scheduled tasks • Microservices  
**❌ Avoid for:** Long-running apps (>15min) • High-frequency predictable loads

**🚀 AWS Lambda: Focus on code, not infrastructure. Scale infinitely, pay precisely.**
