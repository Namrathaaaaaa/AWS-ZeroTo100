# 🌐 AWS CLI - Complete Guide

The **AWS Command Line Interface (AWS CLI)** is a tool that enables you to interact with AWS services using command-line commands. It simplifies automation and scripting of tasks like managing EC2 instances, S3 buckets, IAM users, Lambda functions, and more.

---

## 📦 What is AWS CLI?

- A **unified tool** to manage AWS services.
- Allows access to AWS resources via **terminal or command prompt**.
- Supports scripting and automation.
- Works with **Windows, macOS, and Linux**.
- Communicates with AWS services over **HTTPS** using the AWS SDK.

---

```bash
aws configure
aws s3 mb s3://my-new-bucket
aws s3 ls
aws s3 cp file.txt s3://my-new-bucket/
aws s3 cp s3://my-new-bucket/file.txt .
aws s3 sync ./local-folder s3://my-new-bucket/
aws ec2 describe-instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 create-key-pair --key-name MyKeyPair
aws iam list-users
aws iam create-user --user-name my-user
aws iam attach-user-policy --user-name my-user \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws ec2 describe-vpcs
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws lambda list-functions
aws lambda invoke --function-name my-function out.json
aws configure list
```



