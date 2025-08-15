# AWS Lambda - README

## 📘 What is AWS Lambda?

**AWS Lambda** is a **serverless compute service** that lets you run code without provisioning or managing servers. You simply upload your code, and Lambda automatically handles the compute resources needed to run and scale your application with high availability.

Lambda is event-driven, meaning your code runs in response to events such as:
- HTTP requests via API Gateway
- File uploads to S3
- Database changes in DynamoDB
- Messages in SQS queues
- Scheduled events (cron jobs)

---

## 🚀 Why Use AWS Lambda?

- ✅ **No Server Management**: Focus on code, not infrastructure
- 🔄 **Automatic Scaling**: Scales from zero to thousands of concurrent executions
- 💰 **Pay-per-Use**: Only pay for compute time you consume
- ⚡ **Fast Deployment**: Deploy code changes in seconds
- 🔐 **Built-in Security**: Integrated with AWS IAM and VPC
- 🌍 **Multiple Languages**: Supports Python, Node.js, Java, C#, Go, Ruby, and more

---

## 🏗️ Lambda Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Event     │───▶│   Lambda    │───▶│   Output    │
│  Sources    │    │  Function   │    │Destinations │
└─────────────┘    └─────────────┘    └─────────────┘
     │                    │                   │
  • API Gateway        • Your Code         • DynamoDB
  • S3 Bucket          • Runtime           • S3 Bucket
  • DynamoDB           • Memory            • SQS Queue
  • CloudWatch         • Timeout           • SNS Topic
  • SQS Queue          • Environment       • API Response
  • EventBridge        • Variables         • CloudWatch Logs
```

---

## 🧰 Key Concepts

| Concept | Description |
|---------|-------------|
| **Function** | A piece of code that Lambda executes |
| **Runtime** | The language-specific environment (Python 3.9, Node.js 18, etc.) |
| **Handler** | The entry point method in your code |
| **Event** | JSON data passed to the function when it's invoked |
| **Context** | Runtime information about the Lambda function execution |
| **Layer** | A way to share code and dependencies across multiple functions |
| **Trigger** | An event source that invokes your Lambda function |

---

## 🛠️ Creating Your First Lambda Function

### 1. Using AWS Console

1. **Go to Lambda Console**
   - AWS Console → Lambda → Create Function

2. **Choose Creation Method**
   - Author from scratch
   - Use a blueprint
   - Browse serverless app repository

3. **Basic Information**
   - Function name: `my-first-lambda`
   - Runtime: Python 3.9
   - Execution role: Create a new role with basic Lambda permissions

4. **Write Your Code**
   ```python
   import json
   
   def lambda_handler(event, context):
       # Your code here
       message = f"Hello from Lambda! Received: {json.dumps(event)}"
       
       return {
           'statusCode': 200,
           'body': json.dumps({
               'message': message,
               'timestamp': context.aws_request_id
           })
       }
   ```

### 2. Using AWS CLI

```bash
# Create a deployment package
echo 'import json
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps("Hello from Lambda!")
    }' > lambda_function.py

zip function.zip lambda_function.py

# Create the function
aws lambda create-function \
  --function-name my-first-lambda \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### 3. Using AWS SAM (Serverless Application Model)

**template.yaml**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  MyLambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Runtime: python3.9
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

---

## 💡 Common Use Cases

### 1. **Web APIs and Microservices**
```python
# API Gateway + Lambda for REST API
import json

def lambda_handler(event, context):
    http_method = event['httpMethod']
    path = event['path']
    
    if http_method == 'GET' and path == '/users':
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps([
                {'id': 1, 'name': 'John Doe'},
                {'id': 2, 'name': 'Jane Smith'}
            ])
        }
    
    return {
        'statusCode': 404,
        'body': json.dumps({'error': 'Not Found'})
    }
```

### 2. **File Processing**
```python
# S3 triggered Lambda for image processing
import boto3
from PIL import Image
import io

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Get the object from the event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download the image
    response = s3.get_object(Bucket=bucket, Key=key)
    image = Image.open(io.BytesIO(response['Body'].read()))
    
    # Create thumbnail
    image.thumbnail((200, 200))
    
    # Save thumbnail back to S3
    output_buffer = io.BytesIO()
    image.save(output_buffer, format='JPEG')
    output_buffer.seek(0)
    
    s3.put_object(
        Bucket=bucket,
        Key=f"thumbnails/{key}",
        Body=output_buffer.getvalue()
    )
    
    return {'statusCode': 200, 'body': 'Thumbnail created'}
```

### 3. **Scheduled Tasks**
```python
# CloudWatch Events triggered Lambda for scheduled tasks
import boto3
import json
from datetime import datetime

def lambda_handler(event, context):
    # Daily cleanup task
    s3 = boto3.client('s3')
    
    # Delete old temporary files
    bucket_name = 'my-temp-bucket'
    response = s3.list_objects_v2(
        Bucket=bucket_name,
        Prefix='temp/',
        MaxKeys=1000
    )
    
    if 'Contents' in response:
        for obj in response['Contents']:
            # Delete files older than 7 days
            age = datetime.now().replace(tzinfo=None) - obj['LastModified'].replace(tzinfo=None)
            if age.days > 7:
                s3.delete_object(Bucket=bucket_name, Key=obj['Key'])
                print(f"Deleted: {obj['Key']}")
    
    return {'statusCode': 200, 'body': 'Cleanup completed'}
```

### 4. **Data Processing Pipeline**
```python
# DynamoDB Stream triggered Lambda for data processing
import boto3
import json

def lambda_handler(event, context):
    # Process DynamoDB stream records
    for record in event['Records']:
        event_name = record['eventName']
        
        if event_name == 'INSERT':
            # New item added
            new_item = record['dynamodb']['NewImage']
            process_new_user(new_item)
        
        elif event_name == 'MODIFY':
            # Item updated
            old_item = record['dynamodb']['OldImage']
            new_item = record['dynamodb']['NewImage']
            process_user_update(old_item, new_item)
    
    return {'statusCode': 200}

def process_new_user(user_data):
    # Send welcome email, create profile, etc.
    print(f"Processing new user: {user_data}")

def process_user_update(old_data, new_data):
    # Handle user profile updates
    print(f"User updated from {old_data} to {new_data}")
```

---

## 🔧 Advanced Features

### 1. **Environment Variables**
```python
import os

def lambda_handler(event, context):
    database_url = os.environ.get('DATABASE_URL')
    api_key = os.environ.get('API_KEY')
    
    # Use environment variables in your code
    return {'statusCode': 200}
```

### 2. **Lambda Layers**
```bash
# Create a layer for shared dependencies
mkdir python
pip install requests -t python/
zip -r my-layer.zip python/

# Create layer
aws lambda publish-layer-version \
  --layer-name shared-dependencies \
  --zip-file fileb://my-layer.zip \
  --compatible-runtimes python3.9
```

### 3. **Dead Letter Queues**
```python
# Configure DLQ for failed function executions
import boto3

def lambda_handler(event, context):
    try:
        # Your business logic here
        process_event(event)
        return {'statusCode': 200}
    except Exception as e:
        # Lambda will send failed events to DLQ
        print(f"Error processing event: {str(e)}")
        raise e
```

### 4. **VPC Configuration**
```yaml
# SAM template with VPC configuration
MyLambdaFunction:
  Type: AWS::Serverless::Function
  Properties:
    VpcConfig:
      SecurityGroupIds:
        - sg-12345678
      SubnetIds:
        - subnet-12345678
        - subnet-87654321
```

---

## 🔐 Security Best Practices

### 1. **IAM Roles and Policies**
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
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### 2. **Secrets Management**
```python
import boto3
import json

def get_secret(secret_name):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

def lambda_handler(event, context):
    # Get database credentials from Secrets Manager
    db_credentials = get_secret('prod/database')
    
    # Use credentials safely
    return {'statusCode': 200}
```

### 3. **Resource-Based Policies**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:MyFunction",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "123456789012"
        }
      }
    }
  ]
}
```

---

## 📊 Monitoring and Debugging

### 1. **CloudWatch Logs**
```python
import logging

# Configure logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(f"Received event: {json.dumps(event)}")
    
    try:
        result = process_data(event)
        logger.info(f"Processing successful: {result}")
        return {'statusCode': 200, 'body': result}
    except Exception as e:
        logger.error(f"Error processing event: {str(e)}")
        return {'statusCode': 500, 'body': 'Internal error'}
```

### 2. **X-Ray Tracing**
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch all AWS SDK calls
patch_all()

@xray_recorder.capture('lambda_handler')
def lambda_handler(event, context):
    with xray_recorder.in_subsegment('data_processing'):
        # Your code here
        result = process_data(event)
    
    return {'statusCode': 200, 'body': result}
```

### 3. **Custom Metrics**
```python
import boto3

cloudwatch = boto3.client('cloudwatch')

def lambda_handler(event, context):
    # Send custom metric
    cloudwatch.put_metric_data(
        Namespace='MyApp/Lambda',
        MetricData=[
            {
                'MetricName': 'ProcessedItems',
                'Value': len(event.get('items', [])),
                'Unit': 'Count'
            }
        ]
    )
    
    return {'statusCode': 200}
```

---

## 💰 Cost Optimization

### 1. **Right-Sizing Memory**
```python
# Monitor memory usage in CloudWatch logs
import psutil

def lambda_handler(event, context):
    # Get memory usage
    memory_usage = psutil.virtual_memory().percent
    print(f"Memory usage: {memory_usage}%")
    
    # Your code here
    return {'statusCode': 200}
```

### 2. **Provisioned Concurrency**
```bash
# Configure provisioned concurrency for predictable performance
aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier '$LATEST' \
  --provisioned-concurrency-units 10
```

### 3. **Connection Pooling**
```python
import pymysql

# Create connection pool outside handler (reused across invocations)
connection_pool = pymysql.connect(
    host='database.cluster-xxx.us-east-1.rds.amazonaws.com',
    user='username',
    password='password',
    database='mydb'
)

def lambda_handler(event, context):
    # Reuse existing connection
    cursor = connection_pool.cursor()
    cursor.execute("SELECT * FROM users")
    results = cursor.fetchall()
    
    return {'statusCode': 200, 'body': str(results)}
```

---

## 📈 Performance Best Practices

### 1. **Cold Start Optimization**
```python
# Import modules and initialize connections outside handler
import json
import boto3

# Initialize clients outside handler
s3_client = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')

def lambda_handler(event, context):
    # Handler executes faster on warm starts
    return {'statusCode': 200}
```

### 2. **Async Processing**
```python
import asyncio
import aiohttp

async def fetch_data(session, url):
    async with session.get(url) as response:
        return await response.json()

def lambda_handler(event, context):
    async def main():
        async with aiohttp.ClientSession() as session:
            tasks = [
                fetch_data(session, url) 
                for url in event.get('urls', [])
            ]
            results = await asyncio.gather(*tasks)
            return results
    
    # Run async code
    loop = asyncio.get_event_loop()
    results = loop.run_until_complete(main())
    
    return {'statusCode': 200, 'body': json.dumps(results)}
```

---

## 🧪 Testing Lambda Functions

### 1. **Local Testing**
```python
# test_lambda.py
import json
from lambda_function import lambda_handler

def test_lambda_handler():
    # Create test event
    test_event = {
        'httpMethod': 'GET',
        'path': '/users',
        'queryStringParameters': {'limit': '10'}
    }
    
    # Create mock context
    class MockContext:
        def __init__(self):
            self.aws_request_id = 'test-request-id'
            self.function_name = 'test-function'
    
    context = MockContext()
    
    # Call the handler
    response = lambda_handler(test_event, context)
    
    # Assert results
    assert response['statusCode'] == 200
    assert 'body' in response

if __name__ == '__main__':
    test_lambda_handler()
    print("Test passed!")
```

### 2. **Unit Testing with Moto**
```python
import boto3
import pytest
from moto import mock_s3
from lambda_function import lambda_handler

@mock_s3
def test_s3_processing():
    # Create mock S3 bucket
    s3 = boto3.client('s3', region_name='us-east-1')
    s3.create_bucket(Bucket='test-bucket')
    
    # Upload test file
    s3.put_object(
        Bucket='test-bucket',
        Key='test-file.txt',
        Body=b'test content'
    )
    
    # Create test event
    event = {
        'Records': [{
            's3': {
                'bucket': {'name': 'test-bucket'},
                'object': {'key': 'test-file.txt'}
            }
        }]
    }
    
    # Test the function
    response = lambda_handler(event, None)
    assert response['statusCode'] == 200
```

---

## 🔄 CI/CD for Lambda

### 1. **Using AWS SAM**
```yaml
# buildspec.yml for CodeBuild
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.9
    commands:
      - pip install aws-sam-cli
  
  build:
    commands:
      - sam build
      - sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

### 2. **Using Serverless Framework**
```yaml
# serverless.yml
service: my-lambda-service

provider:
  name: aws
  runtime: python3.9
  region: us-east-1

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
```

### 3. **Blue/Green Deployments**
```yaml
# SAM template with deployment configuration
MyLambdaFunction:
  Type: AWS::Serverless::Function
  Properties:
    AutoPublishAlias: live
    DeploymentPreference:
      Type: Linear10PercentEvery5Minutes
      Hooks:
        PreTraffic: !Ref AliasUpdatePreHook
        PostTraffic: !Ref AliasUpdatePostHook
```

---

## 💰 Pricing

### Free Tier
- **1 million free requests** per month
- **400,000 GB-seconds** of compute time per month

### Paid Usage
- **Requests**: $0.20 per 1 million requests
- **Duration**: $0.0000166667 per GB-second
- **Provisioned Concurrency**: $0.0000041667 per GB-second

### Example Cost Calculation
```
Function: 512 MB memory, runs 100ms, invoked 1 million times/month

Compute Cost:
- GB-seconds: (512/1024) × 0.1 × 1,000,000 = 50,000 GB-seconds
- Cost: 50,000 × $0.0000166667 = $0.83

Request Cost:
- 1 million requests × $0.20 = $0.20

Total: $1.03/month
```

---

## 🆘 Troubleshooting Common Issues

### 1. **Timeout Errors**
```python
# Increase timeout in function configuration
# Optimize code for better performance
import signal

class TimeoutError(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutError("Function timed out")

def lambda_handler(event, context):
    # Set alarm for remaining time
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(int(context.get_remaining_time_in_millis() / 1000) - 1)
    
    try:
        # Your code here
        return {'statusCode': 200}
    except TimeoutError:
        return {'statusCode': 408, 'body': 'Request timeout'}
    finally:
        signal.alarm(0)
```

### 2. **Memory Issues**
```python
import gc
import psutil

def lambda_handler(event, context):
    # Monitor memory usage
    initial_memory = psutil.virtual_memory().percent
    
    try:
        # Process data in chunks
        for chunk in process_in_chunks(event['data']):
            process_chunk(chunk)
            
            # Force garbage collection
            gc.collect()
            
            current_memory = psutil.virtual_memory().percent
            if current_memory > 80:  # 80% threshold
                print(f"High memory usage: {current_memory}%")
    
    except MemoryError:
        return {'statusCode': 500, 'body': 'Out of memory'}
    
    return {'statusCode': 200}
```

### 3. **Cold Start Issues**
```python
# Minimize cold starts
import json

# Initialize expensive operations outside handler
EXPENSIVE_RESOURCE = initialize_resource()

def lambda_handler(event, context):
    # Use pre-initialized resource
    result = EXPENSIVE_RESOURCE.process(event)
    return {'statusCode': 200, 'body': json.dumps(result)}

# Use provisioned concurrency for critical functions
```

---

## 📖 Additional Resources

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [Serverless Framework](https://www.serverless.com/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [AWS Lambda Powertools](https://awslabs.github.io/aws-lambda-powertools-python/)
- [Serverless Patterns Collection](https://serverlessland.com/patterns)

---

**AWS Lambda enables you to build scalable, event-driven applications without managing servers, paying only for the compute time you consume.**
