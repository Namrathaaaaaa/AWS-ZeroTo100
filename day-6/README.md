# AWS Lambda & Serverless Computing

## 🌟 What is Serverless?

**Serverless** = You write code, cloud provider manages infrastructure. Zero server management, auto-scaling, pay-per-execution.

**Benefits:** No servers to manage • Auto-scaling (0 to millions) • Pay only when code runs • Event-driven execution

---

## 📘 AWS Lambda

**AWS Lambda** runs your code in response to events without provisioning servers.

**How it works:** Event triggers → Lambda executes code → You pay for execution time only

**Common triggers:** API Gateway, S3, DynamoDB, SQS, CloudWatch Events, IoT

**Languages:** Python, Node.js, Java, Go, C#, Ruby, PowerShell

---

## 🛠️ Quick Start

### Basic Function

```python
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps(f"Hello {event.get('name', 'World')}!")
    }
```

### Deploy via CLI

```bash
zip function.zip lambda_function.py
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

---

## 💡 Use Cases & Examples

### 1. REST API

```python
def lambda_handler(event, context):
    if event['httpMethod'] == 'GET' and event['path'] == '/users':
        return {
            'statusCode': 200,
            'body': json.dumps([{'id': 1, 'name': 'Alice'}])
        }
    return {'statusCode': 404}
```

### 2. File Processing (S3 trigger)

```python
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    # Process file and save result
    obj = s3.get_object(Bucket=bucket, Key=key)
    processed = obj['Body'].read().decode().upper()
    s3.put_object(Bucket=bucket, Key=f"processed/{key}", Body=processed)

    return {'statusCode': 200}
```

### 3. Scheduled Task

```python
def lambda_handler(event, context):
    # Daily cleanup, backups, etc.
    print("Running scheduled maintenance...")
    return {'statusCode': 200}
```

### 4. IoT Data Processing

```python
import boto3

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('SensorData')

    for record in event['Records']:
        data = json.loads(record['body'])
        table.put_item(Item={
            'deviceId': data['deviceId'],
            'temperature': data['temperature'],
            'alert': 'HIGH' if data['temperature'] > 30 else 'NORMAL'
        })

    return {'statusCode': 200}
```

---

## 🔐 Security & Best Practices

### IAM Role (Minimal Permissions)

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
# Initialize outside handler for reuse
import boto3
s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # Use pre-initialized client
    return {'statusCode': 200}
```

---

## 📊 Architecture Patterns

- **API Backend:** Mobile App → API Gateway → Lambda → Database
- **File Processing:** S3 Upload → Lambda → Process → Store Result
- **Stream Processing:** IoT → Kinesis → Lambda → Analytics
- **Event-Driven:** Database Change → Lambda → Send Notification

---

## 💰 Pricing

- **Free Tier:** 1M requests + 400,000 GB-seconds/month
- **Cost:** ~$1 for 1M requests (512MB, 100ms each)
- **No idle charges** - Pay only for execution time

---

## 🆘 Common Issues

| Issue               | Solution                     |
| ------------------- | ---------------------------- |
| Cold starts         | Use Provisioned Concurrency  |
| Timeout (15min max) | Break into smaller functions |
| Memory errors       | Increase memory allocation   |
| Permission denied   | Check IAM role               |
| Package too large   | Use Lambda Layers            |

---

## 🚀 Deployment

**Console:** Upload zip file  
**CLI:** `aws lambda update-function-code`  
**SAM:** `sam deploy`  
**Serverless:** `serverless deploy`

---

## 🎯 When to Use

**✅ Perfect for:** Event-driven apps, APIs, microservices, data processing, scheduled tasks, MVPs

**❌ Avoid for:** Long-running apps (>15min), persistent connections, complex state management

---

## 📖 Resources

- [AWS Lambda Docs](https://docs.aws.amazon.com/lambda/)
- [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/)
- [Serverless Framework](https://www.serverless.com/)

---

**🚀 AWS Lambda: Write code, not infrastructure. Scale automatically, pay precisely.**
