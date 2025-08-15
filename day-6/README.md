# AWS Lambda & Serverless Computing - README

## 🌟 What is Serverless Computing?

**Serverless** doesn't mean "no servers" - it means you don't manage them. The cloud provider handles all infrastructure while you focus solely on code.

**Key Serverless Benefits:**
- 🚫 **Zero server management** - No provisioning, patching, or scaling servers
- 📈 **Auto-scaling** - Scales from 0 to millions of requests automatically  
- 💰 **Pay-per-execution** - Only pay when code runs, not for idle time
- ⚡ **Event-driven** - Code executes in response to triggers
- 🔧 **Built-in reliability** - High availability and fault tolerance

---

## 📘 AWS Lambda Overview

**AWS Lambda** is Amazon's serverless compute service - the cornerstone of serverless architecture on AWS.

**How it Works:**
1. **Event occurs** → 2. **Lambda provisions resources** → 3. **Code executes** → 4. **Resources released** → 5. **You pay only for execution time**

**Common Triggers:**
- HTTP requests (API Gateway)
- File uploads (S3)
- Database changes (DynamoDB)
- Queue messages (SQS)
- Scheduled events (CloudWatch)
- IoT device data

**Supported Languages:** Python, Node.js, Java, Go, C#, Ruby, PowerShell

---

## 🛠️ Getting Started

### Basic Lambda Function
```python
import json

def lambda_handler(event, context):
    # Process the event
    message = f"Hello! Received: {event.get('name', 'World')}"
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': message,
            'requestId': context.aws_request_id
        })
    }
```

### Deploy via AWS CLI
```bash
# Package your code
zip function.zip lambda_function.py

# Create function
aws lambda create-function \
  --function-name my-serverless-app \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### Using SAM (Serverless Application Model)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  MyServerlessAPI:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Runtime: python3.9
      Events:
        ApiEndpoint:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

---

## 💡 Real-World Use Cases

### 1. Serverless REST API
```python
import json

def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']
    
    if method == 'GET' and path == '/users':
        users = [
            {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'},
            {'id': 2, 'name': 'Bob', 'email': 'bob@example.com'}
        ]
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps(users)
        }
    
    return {'statusCode': 404, 'body': 'Not Found'}
```

### 2. Automated File Processing
```python
import boto3
from PIL import Image
import io

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Get uploaded file details
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Process image (create thumbnail)
    obj = s3.get_object(Bucket=bucket, Key=key)
    image = Image.open(io.BytesIO(obj['Body'].read()))
    image.thumbnail((200, 200))
    
    # Save processed image
    output = io.BytesIO()
    image.save(output, format='JPEG')
    s3.put_object(
        Bucket=bucket,
        Key=f'thumbnails/{key}',
        Body=output.getvalue()
    )
    
    return {'statusCode': 200}
```

### 3. IoT Data Processing
```python
import json
import boto3

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('SensorData')
    
    # Process IoT sensor data
    for record in event['Records']:
        sensor_data = json.loads(record['body'])
        
        # Analyze and store data
        processed_record = {
            'deviceId': sensor_data['deviceId'],
            'temperature': sensor_data['temperature'],
            'timestamp': sensor_data['timestamp'],
            'alert': 'HIGH' if sensor_data['temperature'] > 30 else 'NORMAL'
        }
        
        table.put_item(Item=processed_record)
        
        # Send alert if needed
        if processed_record['alert'] == 'HIGH':
            send_temperature_alert(processed_record)
    
    return {'statusCode': 200}
```

### 4. Scheduled Maintenance Tasks
```python
import boto3
from datetime import datetime, timedelta

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Clean up old log files daily
    cutoff_date = datetime.now() - timedelta(days=30)
    
    response = s3.list_objects_v2(Bucket='app-logs', Prefix='logs/')
    
    deleted_count = 0
    for obj in response.get('Contents', []):
        if obj['LastModified'].replace(tzinfo=None) < cutoff_date:
            s3.delete_object(Bucket='app-logs', Key=obj['Key'])
            deleted_count += 1
    
    return {
        'statusCode': 200,
        'body': f'Cleaned up {deleted_count} old log files'
    }
```

---

## 🔐 Security & Best Practices

### IAM Role (Principle of Least Privilege)
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

### Performance Optimization
```python
import boto3
import os

# Initialize outside handler (connection reuse)
s3_client = boto3.client('s3')
DB_TABLE = os.environ['TABLE_NAME']

def lambda_handler(event, context):
    # Use pre-initialized clients for better performance
    # Environment variables for configuration
    return {'statusCode': 200}
```

---

## 📊 Serverless Architecture Patterns

### 1. Event-Driven Microservices
```
User Upload → S3 → Lambda → DynamoDB → SNS → Email Notification
```

### 2. API-First Architecture  
```
Mobile App → API Gateway → Lambda → RDS/DynamoDB → Response
```

### 3. Stream Processing
```
IoT Devices → Kinesis → Lambda → Analytics → Dashboard
```

---

## 💰 Cost & Pricing

**AWS Lambda Pricing:**
- **Free Tier**: 1M requests + 400,000 GB-seconds/month
- **Requests**: $0.20 per 1M requests
- **Duration**: $0.0000166667 per GB-second

**Cost Example:**
- Function: 512MB memory, 100ms duration
- 1M monthly executions = ~$1.03/month

**Cost Benefits:**
- No infrastructure costs
- No idle time charges
- Automatic scaling without over-provisioning

---

## 🚀 Deployment Options

### 1. AWS Console (Quick Testing)
Upload zip file directly through web interface

### 2. AWS CLI (Automation)
```bash
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip
```

### 3. AWS SAM (Infrastructure as Code)
```bash
sam build
sam deploy --guided
```

### 4. Serverless Framework
```bash
serverless deploy
```

---

## 🆘 Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **Cold Starts** | Function not used recently | Use Provisioned Concurrency for critical functions |
| **Timeout (15min max)** | Long-running processes | Break into smaller functions or use Step Functions |
| **Memory Errors** | Insufficient allocation | Monitor CloudWatch metrics, increase memory |
| **Permission Denied** | Missing IAM permissions | Review and update IAM role policies |
| **Deployment Size** | Package too large (50MB) | Use Lambda Layers for dependencies |

---

## 🧪 Testing Serverless Functions

### Local Testing
```python
# test_lambda.py
import json
from lambda_function import lambda_handler

def test_api_endpoint():
    event = {
        'httpMethod': 'GET',
        'path': '/users',
        'headers': {},
        'body': None
    }
    
    response = lambda_handler(event, {})
    assert response['statusCode'] == 200
    
    print("✅ Test passed!")

if __name__ == '__main__':
    test_api_endpoint()
```

### Integration Testing
```bash
# Using SAM for local testing
sam local start-api
sam local invoke MyFunction --event test-event.json
```

---

## 📖 Essential Resources

- **Documentation**: [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- **Framework**: [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/)
- **Tools**: [Serverless Framework](https://www.serverless.com/)
- **Best Practices**: [AWS Well-Architected Serverless](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/)

---

## 🎯 When to Use Serverless

**Perfect For:**
- ✅ Event-driven applications
- ✅ Microservices architectures  
- ✅ APIs with variable traffic
- ✅ Data processing pipelines
- ✅ Scheduled tasks
- ✅ Prototyping and MVPs

**Consider Alternatives For:**
- ❌ Long-running applications (>15 minutes)
- ❌ High-frequency, predictable workloads
- ❌ Applications requiring persistent connections
- ❌ Complex state management

---

**🚀 AWS Lambda: The future of computing is serverless. Focus on code, not infrastructure. Scale infinitely, pay precisely.**
