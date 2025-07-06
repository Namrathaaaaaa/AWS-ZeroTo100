# AWS CodeCommit - README

## 📘 What is AWS CodeCommit?

**AWS CodeCommit** is a fully managed **source control service** that hosts secure Git-based repositories. It works like GitHub or Bitbucket but runs entirely on AWS, making it ideal for teams already using AWS services.

---

## 🚀 Why Use CodeCommit?

- ✅ Fully managed, no servers to provision or scale.
- 🔐 Secure by default – data is encrypted in transit and at rest.
- 🌍 Supports Git commands you already know.
- 🔗 Integrates with AWS IAM for fine-grained access control.
- 🔄 Seamless integration with AWS CodePipeline, CodeBuild, CodeDeploy.

---

## 🏗️ Typical Use Cases

- Hosting Git repositories in a secure, private cloud environment.
- Building CI/CD pipelines with AWS developer tools.
- Managing source code for Lambda functions, container apps, microservices, and more.

---

## 🧱 Basic Concepts

| Concept        | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| Repository     | A Git-based repo to store your code.                                        |
| Commit         | A snapshot of your code at a point in time.                                 |
| Branch         | A version of your repo used for development or testing.                     |
| Clone/Push/Pull| Standard Git commands to interact with your repo.                           |
| IAM Policies   | Used to control access to repos (read/write/admin).                         |

---

## 🛠️ How to Use AWS CodeCommit

### 1. **Create a Repository**
Go to the AWS Console → CodeCommit → Create Repository → Give it a name and optional description.

### 2. **Set Up Git Credentials**
If not already configured:
- Go to IAM → Your user → Security Credentials
- Generate HTTPS Git credentials

### 3. **Clone the Repository**

```bash
git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/<repository-name>
cd <repository-name>
# Step 1: Create a new file
echo "# Sample AWS CodeCommit Project" > README.md

# Step 2: Initialize Git (if not already initialized)
git init

# Step 3: Add files to staging area
git add .

# Step 4: Commit the changes
git commit -m "Initial commit to CodeCommit"

# Step 5: Add CodeCommit as a remote (if not already added)
git remote add origin https://git-codecommit.<region>.amazonaws.com/v1/repos/<repository-name>

# Step 6: Push to the main branch
git push -u origin main
````