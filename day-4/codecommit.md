# 🚀 AWS CI/CD Pipeline with CodeCommit, CodePipeline, CodeBuild, and CodeDeploy

## 📘 What is AWS CI/CD?

**AWS CI/CD** is a complete DevOps solution using four core AWS services to automate your software delivery pipeline:

- **CodeCommit** - Fully managed Git repository service
- **CodePipeline** - Continuous integration and delivery orchestration
- **CodeBuild** - Managed build service that compiles and tests code
- **CodeDeploy** - Automated deployment service to various AWS compute services

---

## 🚀 Why Use AWS CI/CD Pipeline?

- ✅ **Fully Managed**: No infrastructure to maintain or scale
- 🔐 **Secure**: Integrated with AWS IAM and encrypted by default
- 🔄 **Automated**: Trigger builds and deployments automatically
- 🌍 **Scalable**: Handle projects of any size
- 🔗 **Integrated**: Seamless integration between all AWS services
- 💰 **Cost-Effective**: Pay only for what you use

---

## 🏗️ CI/CD Pipeline Architecture

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌─────────────┐
│ CodeCommit  │───▶│ CodePipeline │───▶│ CodeBuild   │───▶│ CodeDeploy  │
│ (Source)    │    │ (Orchestrate)│    │ (Build/Test)│    │ (Deploy)    │
└─────────────┘    └──────────────┘    └─────────────┘    └─────────────┘
```

---

## 🧰 Prerequisites

- AWS account with appropriate permissions
- IAM user/role with access to:
  - CodeCommit
  - CodePipeline
  - CodeBuild
  - CodeDeploy
  - EC2 (for deployment targets)
  - S3 (for artifacts)
- Git installed locally
- Application code ready for deployment

---

## 1️⃣ AWS CodeCommit - Source Control

### What is CodeCommit?

A fully managed Git repository service that hosts secure, scalable repositories.

### Setup CodeCommit Repository

#### Using AWS Console

1. Go to AWS Console → CodeCommit → Create Repository
2. Enter repository name: `my-app-repo`
3. Add description and click "Create"

#### Using AWS CLI

```bash
aws codecommit create-repository \
  --repository-name my-app-repo \
  --repository-description "My CI/CD application repository"
```

### Configure Git Credentials

```bash
# Set up HTTPS Git credentials in IAM
# Then clone the repository
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-app-repo
cd my-app-repo

# Add your application code
echo "# My Application" > README.md
git add .
git commit -m "Initial commit"
git push origin main
```

---

## 2️⃣ AWS CodeBuild - Build and Test

### What is CodeBuild?

A fully managed build service that compiles source code, runs tests, and produces deployment artifacts.

### Create BuildSpec File

Create `buildspec.yml` in your repository root:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18 # or python: 3.9, java: corretto11, etc.
    commands:
      - echo Installing dependencies...
      - npm install # or pip install -r requirements.txt

  pre_build:
    commands:
      - echo Logging in to Amazon ECR... (if using Docker)
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com

  build:
    commands:
      - echo Build started on `date`
      - echo Running tests...
      - npm test # or python -m pytest
      - echo Building the application...
      - npm run build # or your build command

  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing Docker image... (if applicable)

artifacts:
  files:
    - "**/*"
  name: my-app-artifacts
```

### Create CodeBuild Project

```bash
aws codebuild create-project \
  --name my-app-build \
  --source type=CODECOMMIT,location=https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-app-repo \
  --artifacts type=S3,location=my-build-artifacts-bucket \
  --environment type=LINUX_CONTAINER,image=aws/codebuild/amazonlinux2-x86_64-standard:3.0,computeType=BUILD_GENERAL1_MEDIUM \
  --service-role arn:aws:iam::123456789012:role/CodeBuildServiceRole
```

---

## 3️⃣ AWS CodeDeploy - Automated Deployment

### What is CodeDeploy?

A deployment service that automates application deployments to EC2 instances, on-premises servers, Lambda functions, or ECS services.

### Create Application and Deployment Group

```bash
# Create CodeDeploy application
aws deploy create-application \
  --application-name my-app \
  --compute-platform Server  # or Lambda, ECS

# Create deployment group
aws deploy create-deployment-group \
  --application-name my-app \
  --deployment-group-name my-app-deployment-group \
  --service-role-arn arn:aws:iam::123456789012:role/CodeDeployServiceRole \
  --ec2-tag-filters Key=Environment,Value=Production,Type=KEY_AND_VALUE
```

### Create AppSpec File

Create `appspec.yml` for EC2/On-premises deployments:

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

---

## 4️⃣ AWS CodePipeline - Orchestration

### What is CodePipeline?

A continuous integration and continuous delivery service that orchestrates your entire release pipeline.

### Create Pipeline

```bash
# Create pipeline using AWS CLI (complex JSON structure)
# Or use AWS Console for easier setup
```

### Pipeline Configuration (Console Steps)

1. **Pipeline Settings**

   - Pipeline name: `my-app-pipeline`
   - Service role: Create new or use existing
   - Artifact store: S3 bucket for storing artifacts

2. **Source Stage**

   - Source provider: AWS CodeCommit
   - Repository: `my-app-repo`
   - Branch: `main`
   - Change detection: Amazon CloudWatch Events

3. **Build Stage**

   - Build provider: AWS CodeBuild
   - Project name: `my-app-build`
   - Build type: Single build

4. **Deploy Stage**
   - Deploy provider: AWS CodeDeploy
   - Application name: `my-app`
   - Deployment group: `my-app-deployment-group`

---

## 🔧 Complete Pipeline Example

### 1. Directory Structure

```
my-app-repo/
├── README.md
├── buildspec.yml
├── appspec.yml
├── src/
│   ├── index.js
│   └── package.json
└── scripts/
    ├── install_dependencies.sh
    ├── start_server.sh
    └── stop_server.sh
```

### 2. Sample Application (Node.js)

**package.json**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js",
    "test": "jest",
    "build": "echo 'Build completed'"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

**index.js**

```javascript
const express = require("express");
const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.json({ message: "Hello from AWS CI/CD Pipeline!" });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### 3. Deployment Scripts

**scripts/install_dependencies.sh**

```bash
#!/bin/bash
yum update -y
yum install -y nodejs npm
```

**scripts/start_server.sh**

```bash
#!/bin/bash
cd /var/www/html
npm install
pm2 start index.js --name my-app
```

**scripts/stop_server.sh**

```bash
#!/bin/bash
pm2 stop my-app || true
```

---

## � IAM Roles and Permissions

### CodeBuild Service Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "codecommit:GitPull",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "*"
    }
  ]
}
```

### CodeDeploy Service Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "autoscaling:*",
        "elasticloadbalancing:*",
        "s3:GetObject"
      ],
      "Resource": "*"
    }
  ]
}
```

### CodePipeline Service Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codecommit:GetBranch",
        "codecommit:GetCommit",
        "codebuild:BatchGetBuilds",
        "codebuild:StartBuild",
        "codedeploy:CreateDeployment",
        "codedeploy:GetApplication",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 🚀 Pipeline Workflow

1. **Developer pushes code** to CodeCommit repository
2. **CloudWatch Events** detect the change and trigger CodePipeline
3. **CodePipeline** downloads source code from CodeCommit
4. **CodeBuild** runs the build process using `buildspec.yml`
5. **Build artifacts** are stored in S3
6. **CodeDeploy** deploys the application using `appspec.yml`
7. **Application** is running on target infrastructure

---

## 🎯 Best Practices

### Source Control

- Use meaningful commit messages
- Implement branch protection rules
- Use pull requests for code reviews
- Tag releases for version tracking

### Build Process

- Keep build times minimal
- Run comprehensive tests
- Use build caching when possible
- Store build artifacts securely

### Deployment

- Use blue/green deployments for zero downtime
- Implement automatic rollback on failure
- Monitor application health during deployment
- Use deployment configuration for gradual rollouts

### Pipeline Management

- Use separate pipelines for different environments
- Implement manual approval gates for production
- Monitor pipeline execution with CloudWatch
- Set up notifications for pipeline events

---

## 💰 Pricing Overview

| Service      | Pricing                       |
| ------------ | ----------------------------- |
| CodeCommit   | $1/user/month + storage costs |
| CodeBuild    | $0.005/build minute           |
| CodeDeploy   | Free for EC2/On-premises      |
| CodePipeline | $1/pipeline/month             |

---

## 🆘 Troubleshooting Common Issues

### Build Failures

- Check buildspec.yml syntax
- Verify IAM permissions for CodeBuild role
- Review build logs in CloudWatch
- Ensure dependencies are available

### Deployment Failures

- Verify appspec.yml configuration
- Check EC2 instance health and CodeDeploy agent
- Review deployment logs
- Ensure proper IAM permissions

### Pipeline Issues

- Check service role permissions
- Verify S3 bucket access
- Review CloudWatch Events configuration
- Check artifact passing between stages

---

## 📖 Additional Resources

- [AWS CodeCommit User Guide](https://docs.aws.amazon.com/codecommit/)
- [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/)
- [AWS CodeBuild User Guide](https://docs.aws.amazon.com/codebuild/)
- [AWS CodeDeploy User Guide](https://docs.aws.amazon.com/codedeploy/)
- [AWS CI/CD Workshop](https://cicd.serverlessworkshops.io/)

---

**Build robust, scalable CI/CD pipelines with AWS Developer Tools for faster, more reliable software delivery.**
