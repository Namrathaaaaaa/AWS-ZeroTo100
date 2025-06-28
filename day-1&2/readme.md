# ☁️ Introduction to Cloud Computing and AWS IAM 🔐

## 🕰️ What Is Cloud & A Brief History

Before the rise of cloud computing (over 20 years ago), deploying an application required buying **physical servers** located in various regions. This process included:

- Purchasing expensive hardware
- Manually configuring networking
- Handling maintenance and scaling issues

💡 **Problem:** Suppose your app needed only 1 GB RAM — you'd still have to buy a server with 100 GB RAM, leading to **wasted resources** and **high maintenance costs**.

To solve this, **AWS (Amazon Web Services)** introduced cloud computing by offering:
- On-demand infrastructure
- Pay-as-you-go pricing
- Fully managed services

This innovation eliminated the need for physical server ownership and made deployment easier, faster, and cost-effective.

---

## 🆚 Public Cloud vs Private Cloud

### 🔒 Private Cloud
- Infrastructure is **owned and managed** by an organization.
- Servers are **physically maintained** on-premises.
- Full control, but higher costs and more complexity.

### ☁️ Public Cloud
- Infrastructure is **owned by a cloud provider** (like AWS, Azure, GCP).
- Users rent servers and services on demand.
- **Highly scalable**, cost-effective, and easy to manage.

---

## 💡 Why Public Cloud Is Better

1. ✅ **Lower Cost** – No need to purchase and maintain hardware.
2. ✅ **Less Overhead** – Cloud providers handle infrastructure, networking, backups, and security.
3. ✅ **Faster Time to Deploy** – Scale up/down in minutes.

---

## 🏆 Why AWS Is the Leading Cloud Provider

1. **First Mover Advantage** – AWS was the first major cloud provider.
2. **Market Leadership** – AWS has the largest share in the cloud market.
3. **Rich Ecosystem** – Offers 200+ services across compute, storage, AI, security, DevOps, and more.
4. **Global Reach** – Data centers in multiple regions and availability zones.

---

# 🔐 AWS IAM – Identity and Access Management

## 📌 What Is AWS IAM?

**AWS IAM** is a secure web service that helps manage **who can access** AWS resources and **what actions** they can perform.

IAM allows fine-grained access control to ensure:
- 🔑 Only authorized users can access resources
- 📋 Permissions are granted using policies
- 🎯 Resources are protected with least privilege access

---

## 👤 IAM Users

- Represents a **person or application** with permanent credentials.
- Each user is created under your AWS account.
- Access types:
  - **Console access** (username + password)
  - **Programmatic access** (access key + secret key)

> 🧠 Think of IAM users like individual login identities for people or apps.

---

## 👥 IAM Groups

- A **collection of IAM users**.
- You can attach policies to groups to apply permissions to all members.
- Common groups:
  - `Admins`, `Developers`, `ReadOnlyUsers`

> 🧠 Use groups to **manage permissions at scale**.

---

## 📜 IAM Policies

- Policies define **what actions are allowed or denied** on AWS resources.
- Written in **JSON format**.
- Can be attached to:
  - **Users**
  - **Groups**
  - **Roles**

### Types of Policies
- ✅ **AWS Managed Policies** – Predefined and maintained by AWS.
- ✍️ **Customer Managed Policies** – Custom-created for specific needs.

### 📦 Example Policy
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



