# AWS ECR (Elastic Container Registry)

## 📘 What is AWS ECR?

**AWS ECR** is a fully managed Docker container registry for storing, managing, and deploying container images. Think Docker Hub, but on AWS with enterprise security.

**Benefits:** Fully managed • Secure (IAM) • Scalable • AWS integration • Pay-per-use

---

## 🛠️ Quick Start

```bash
# 1. Create repository
aws ecr create-repository --repository-name my-app

# 2. Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# 3. Build, tag, and push
docker build -t my-app .
docker tag my-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# 4. Pull image
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```

---

## 💡 Use Cases

### CI/CD Pipeline
```bash
# In your build script
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
docker build -t $ECR_REGISTRY/my-app:$BUILD_ID .
docker push $ECR_REGISTRY/my-app:$BUILD_ID
```

### ECS/Fargate Deployment
```json
{
  "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
  "memory": 512,
  "portMappings": [{"containerPort": 80}]
}
```

### Lambda Container Functions
```bash
# Push Lambda container image
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-lambda:latest

# Create Lambda function
aws lambda create-function \
  --function-name my-function \
  --package-type Image \
  --code ImageUri=123456789012.dkr.ecr.us-east-1.amazonaws.com/my-lambda:latest
```

---

## 🔐 Security & Management

### Basic IAM Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer",
        "ecr:PutImage"
      ],
      "Resource": "*"
    }
  ]
}
```

### Essential Commands
```bash
# List repositories
aws ecr describe-repositories

# List images
aws ecr list-images --repository-name my-app

# Enable image scanning
aws ecr put-image-scanning-configuration --repository-name my-app --image-scanning-configuration scanOnPush=true

# Delete old images
aws ecr batch-delete-image --repository-name my-app --image-ids imageTag=old-version
```

---

## 📊 Key Features

**Image Scanning:** Automatic vulnerability detection  
**Lifecycle Policies:** Auto-delete old images  
**Cross-Account Access:** Share images between AWS accounts  
**Regional Replication:** Copy images across regions  

---

## 💰 Pricing

- **Storage:** $0.10 per GB/month
- **Transfer:** Free within AWS, $0.09/GB to internet
- **Example:** 10GB storage = $1/month

---

## 🆘 Common Issues

| Issue | Solution |
|-------|----------|
| Authentication failed | Run `aws ecr get-login-password` |
| Permission denied | Check IAM ECR permissions |
| Image not found | Verify repository name and tag |
| Push timeout | Check image size and network |

---

## 🎯 When to Use

**✅ Use ECR for:** AWS deployments, ECS/EKS, Lambda containers, enterprise security  
**❌ Avoid for:** Multi-cloud, open source projects, simple dev setups

---

## 📖 Resources

- [ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/)

---

**🚀 AWS ECR: Secure container registry that integrates seamlessly with AWS services.**