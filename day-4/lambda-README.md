# AWS Lambda - README

## 📘 What is AWS Lambda?

**AWS Lambda** is a serverless compute service that runs your code in response to events without managing servers. You pay only for execution time.

**Key Benefits:**
- 🚫 No server management
- 📈 Automatic scaling (0 to millions)
- 💰 Pay-per-use pricing
- ⚡ Event-driven execution

---

## 🛠️ Quick Start

### Basic Function
```python
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

### Create via CLI
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

## 💡 Common Use Cases

### 1. API Backend
```python
def lambda_handler(event, context):
    if event['httpMethod'] == 'GET':
        return {
            'statusCode': 200,
            'body': json.dumps([{'id': 1, 'name': 'John'}])
        }
    return {'statusCode': 404}
```

### 2. File Processing
```python
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Process file
    obj = s3.get_object(Bucket=bucket, Key=key)
    processed_content = obj['Body'].read().decode().upper()
    
    # Save result
    s3.put_object(Bucket=bucket, Key=f"processed/{key}", Body=processed_content)
    return {'statusCode': 200}
```

### 3. Scheduled Task
```python
def lambda_handler(event, context):
    # Daily cleanup or backup task
    print("Running scheduled task...")
    return {'statusCode': 200}
```

---

## 🔐 IAM Role (Basic)
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
      "Resource": "*"
    }
  ]
}
```

---

## 💰 Pricing
- **Free Tier**: 1M requests + 400,000 GB-seconds/month
- **Cost**: ~$1 for 1M requests (512MB, 100ms each)

---

## 🆘 Common Issues
| Issue | Solution |
|-------|----------|
| Timeout | Increase timeout in settings |
| Memory | Monitor usage, increase allocation |
| Permissions | Check IAM role permissions |

---

## 📖 Resources
- [AWS Lambda Guide](https://docs.aws.amazon.com/lambda/)
- [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/)

---

**AWS Lambda: Serverless computing made simple. Write code, deploy, scale automatically.**
