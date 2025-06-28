# What is cloud and history: 
Before the past 20 years there is no cloud platforms, so whenever we required to deploy the application, we have to buy the servers physically from different regions of the world and have to configure its network and deploy the application in to those servers. After deploying suppose we require only 1gb of ram for maintaining, we don't have an option to buy only 1gb ram servers. We have to buy 100gb of ram servers. From this the major drawback is there is lot of waste of ram usage and more cost to maintenance servers physically and it is difficult overall to maintain physical servers. So, to encounter this problem AWS come forward and introduced its cloud services by giving only required amount of servers to the users and paying for only whatever they have used. 

# Public vs Private cloud :
Private cloud is simply buying servers physically and deploying our applications without the help of any cloud providers. 
On the other hand Public cloud is the cloud where we can buy servers from the cloud providers.

# Why Public cloud is better than private?
1. Less cost compare to private cloud.
2. Public cloud cuts down the major headache to their users such as maintaining servers and checking connections and giving network configurations as everything will be taken by cloud providers only. 

# Why AWS is better than others?
1. AWS is the first cloud provider in the industry, it has first come advantage.
2. AWS have the majority of the market share and also they have more services. So, many companies are preferring AWS over other cloud providers. 


# AWS IAM – Identity and Access Management 🔐

This README provides a brief overview of **AWS IAM**, focusing on the core components: **Users**, **Policies**, **Groups**, and **Roles**.

---

## 📌 What is AWS IAM?

**AWS IAM (Identity and Access Management)** is a web service that helps you securely control access to AWS services and resources. IAM allows you to manage:

- **Who** can access your AWS environment
- **What** actions they can perform
- **Which** resources they can interact with

---

## 👤 IAM Users

- An **IAM User** represents a single person or application.
- Each user has a unique name within the AWS account.
- Users can be assigned:
  - **Programmatic access** (via Access Key ID & Secret Access Key)
  - **Console access** (username + password for AWS Management Console)

> 🧠 Think of an IAM user like a login identity for a person or script.

---

## 👥 IAM Groups

- An **IAM Group** is a collection of IAM users.
- Groups let you apply **policies** to multiple users at once.
- Common examples:
  - `Admins`, `Developers`, `ReadOnlyUsers`

> 🧠 Groups are for managing permissions at scale — users inherit the group’s policies.

---

## 📜 IAM Policies

- A **policy** is a JSON document that defines **what actions** are allowed or denied.
- Policies are attached to:
  - **Users**
  - **Groups**
  - **Roles**
- Two types of policies:
  - **AWS Managed Policies** – Predefined by AWS
  - **Customer Managed Policies** – Created by you

Example of a simple policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}

# 🎭 AWS IAM Roles – Quick Overview

## What is an IAM Role?
An **IAM Role** is a temporary identity in AWS that provides specific permissions and is assumed by trusted entities like AWS services, IAM users, or external identities. Unlike users, roles do not have permanent credentials.

## Why Use Roles?
- Grant **temporary access** to AWS resources
- Enable **services like EC2/Lambda** to interact with other AWS services
- Allow **cross-account** or **federated access** securely

## Key Components:
- **Trust Policy**: Defines who can assume the role (e.g., EC2)
- **Permissions Policy**: Defines what actions the role allows

## Example Use Cases:
- EC2 instance accessing S3
- Granting admin access to a user for a short period
- Enabling SSO via Google or Active Directory

## Best Practices:
- Follow **least privilege principle**
- Monitor role usage
- Use **CloudTrail** for auditing

> IAM Roles = Secure, flexible, temporary access across AWS.

