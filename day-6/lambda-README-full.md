# AWS Lambda - README

## 🌟 Introduction to Serverless Computing

**Serverless computing** is a cloud execution model where you focus solely on writing and deploying code while the cloud provider manages all underlying infrastructure. You don't eliminate servers—you just don't manage them!

**Key Benefits:**
- 🚫 **No Server Management** - Focus on code, not infrastructure
- 📈 **Automatic Scaling** - From zero to millions of requests
- � **Pay-per-Use** - Only pay when code runs
- ⚡ **Event-Driven** - Code executes in response to events
- 🔧 **Built-in Resilience** - High availability by default

---

## 📘 Understanding AWS Lambda

**AWS Lambda** is Amazon's serverless compute service that runs your code in response to events without provisioning or managing servers. It's the cornerstone of serverless architecture on AWS.

**How Lambda Functions Work:**
1. **Event occurs** (file upload, API request, timer, etc.)
2. **Lambda automatically provisions** compute resources
3. **Your code executes** in an isolated environment
4. **Resources are released** when execution completes
5. **You pay only** for the execution time used

**Key Triggers:**
**Supported Languages:**
- Python, Node.js, Java, Go, C#, Ruby, PowerShell
- Custom runtimes for any programming language

---

## 🚀 Why Choose Lambda?

- ✅ **Event-Driven Execution** - Functions triggered by specific events
- 🔄 **Automatic Scaling** - Handle 1 user or 1 million users seamlessly  
- 💰 **Cost Efficiency** - Pay only for compute time consumed
- ⚡ **Fast Deployment** - Deploy code changes in seconds
- 🔐 **Built-in Security** - IAM integration and VPC support
- 🌍 **Multi-Language Support** - Use your preferred programming language

---

## 🎯 Real-World Use Cases

### 1. **Automated Image Processing**
Perfect for photo-sharing apps where users upload images daily.
```python
# Automatically resize/compress images when uploaded to S3
import boto3
from PIL import Image
import io

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download and process image
    obj = s3.get_object(Bucket=bucket, Key=key)
    image = Image.open(io.BytesIO(obj['Body'].read()))
    image.thumbnail((200, 200))  # Resize
    
    # Save processed image
    output = io.BytesIO()
    image.save(output, format='JPEG', quality=85)
    s3.put_object(
        Bucket=bucket,
        Key=f"thumbnails/{key}",
        Body=output.getvalue()
    )
    return {'statusCode': 200, 'body': 'Image processed'}
```

### 2. **Chatbots and Virtual Assistants**
Build interactive chatbots for customer service or smart home control.
```python
# Simple chatbot handler
def lambda_handler(event, context):
    user_message = event.get('message', '').lower()
    
    if 'weather' in user_message:
        response = get_weather_info()
    elif 'help' in user_message:
        response = "I can help with weather, schedules, and general questions!"
    else:
        response = "I'm sorry, I didn't understand. Try asking about weather or say 'help'."
    
    return {
        'statusCode': 200,
        'body': {'response': response}
    }
```

### 3. **Scheduled Data Backups**
Automate data backups for disaster recovery and data resilience.
```python
# Daily backup scheduler
import boto3
from datetime import datetime

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Copy data from primary to backup bucket
    source_bucket = 'primary-data'
    backup_bucket = 'backup-data'
    
    # List and copy all objects
    objects = s3.list_objects_v2(Bucket=source_bucket)
    
    for obj in objects.get('Contents', []):
        copy_source = {'Bucket': source_bucket, 'Key': obj['Key']}
        backup_key = f"backup-{datetime.now().strftime('%Y%m%d')}/{obj['Key']}"
        
        s3.copy_object(
            CopySource=copy_source,
            Bucket=backup_bucket,
            Key=backup_key
        )
    
    return {'statusCode': 200, 'body': 'Backup completed'}
```

### 4. **Real-Time Analytics**
Process streaming data from IoT devices or social media for instant insights.
```python
# IoT data processor
import json
import boto3

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('IoTAnalytics')
    
    for record in event['Records']:
        # Parse IoT sensor data
        sensor_data = json.loads(record['body'])
        
        # Process and analyze data
        processed_data = {
            'device_id': sensor_data['device_id'],
            'temperature': sensor_data['temperature'],
            'humidity': sensor_data['humidity'],
            'timestamp': sensor_data['timestamp'],
            'alert': 'HIGH_TEMP' if sensor_data['temperature'] > 30 else 'NORMAL'
        }
        
        # Store analytics results
        table.put_item(Item=processed_data)
        
        # Send alerts if needed
        if processed_data['alert'] == 'HIGH_TEMP':
            send_alert_notification(processed_data)
    
    return {'statusCode': 200}
```

### 5. **API Backends**
Build scalable API backends for web and mobile applications.
```python
# Scalable API backend
import json
import boto3

def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']
    
    if method == 'GET' and path == '/users':
        return get_users()
    elif method == 'POST' and path == '/users':
        return create_user(json.loads(event['body']))
    elif method == 'GET' and path.startswith('/users/'):
        user_id = path.split('/')[-1]
        return get_user(user_id)
    
    return {
        'statusCode': 404,
        'body': json.dumps({'error': 'Endpoint not found'})
    }

def get_users():
    # Fetch users from database
    return {
        'statusCode': 200,
        'body': json.dumps([
            {'id': 1, 'name': 'John Doe', 'email': 'john@example.com'},
            {'id': 2, 'name': 'Jane Smith', 'email': 'jane@example.com'}
        ])
    }
```

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
AWSTemplateFormatVersion: '2010-09-09'
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

## 💡 Additional Use Cases

### Traditional Web Development
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

### Simple File Processing
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

### Basic Scheduled Task
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
    - '**/*'
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
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
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

| Issue | Solution |
|-------|----------|
| **Timeout** | Increase timeout setting, optimize code |
| **Memory** | Monitor usage, increase allocation if needed |
| **Cold starts** | Use provisioned concurrency, optimize imports |
| **Permissions** | Check IAM roles and resource policies |

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

**AWS Lambda: The heart of serverless computing. Focus on code, not servers. Scale automatically from zero to millions of requests. Pay only for what you use.**
