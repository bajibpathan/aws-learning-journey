# 🧪 Lab 04 - Configure AWS CLI Authentication for an IAM User

## 📖 Lab Information

| Property | Value |
|----------|-------|
| **Category** | AWS Identity & Access Management |
| **Level** | 🟡 Intermediate |
| **Estimated Time** | 20–30 Minutes |
| **AWS Services** | IAM, AWS CLI |

---

## 🎯 Objective

Configure the AWS Command Line Interface (AWS CLI) to securely authenticate an IAM User and access AWS services using a named profile.

---

## 📋 Prerequisites

- AWS Account
- Administrator permissions
- AWS CLI installed on the local machine
- IAM User with programmatic access (Access Key)

---

## ⚙️ Implementation Steps

1. Install the AWS CLI on your local machine.
2. Create an IAM User (or use an existing IAM User).
3. Generate an Access Key for the IAM User.
4. Download the Access Key ID and Secret Access Key.
5. Attempt to run an AWS CLI command before configuration.
   - **Expected Result:** Authentication fails because the CLI has not been configured.
6. Configure a named AWS CLI profile:

```bash
aws configure --profile baji-dev
```

7. Enter the following information:
   - AWS Access Key ID
   - AWS Secret Access Key
   - Default Region
   - Default Output Format

8. Verify the configuration by running AWS CLI commands using the named profile.

---

## ✅ Validation

Verify the following:

- The AWS CLI authenticates successfully using the configured profile.
- The IAM User can access only the AWS resources permitted by the attached IAM Policies.
- Commands execute successfully without authentication errors.

---

## 💻 Commands Used

### Configure a Named Profile

```bash
aws configure --profile baji-dev
```

### Verify S3 Access

```bash
aws s3 ls --profile baji-dev
```

### Create an S3 Bucket

```bash
aws s3 mb s3://unique-bucket-name --region us-east-1 --profile baji-dev
```

### Verify the Authenticated Identity

```bash
aws sts get-caller-identity --profile baji-dev
```

---

## 🎯 Result

Successfully configured AWS CLI authentication using a named profile and verified that the IAM User could securely access authorized AWS resources from the command line.

---

## 💡 Lessons Learned

- AWS CLI must be configured before authenticated commands can be executed.
- Named profiles simplify working with multiple AWS accounts.
- IAM Policies determine what actions an authenticated user can perform.
- Authentication confirms identity, while IAM Policies control authorization.

---

## ❓ Frequently Asked Question

### Why use a named profile instead of the default profile?

Named profiles allow you to manage multiple AWS accounts or IAM Users from the same workstation without repeatedly reconfiguring the AWS CLI.

Example:

```text
default
dev
test
prod
sandbox
```

Each profile stores its own credentials and configuration, making it easier to switch between environments.

---

## 🔗 Related Documentation

- [IAM Overview](../README.md)
- [Accessing AWS](../README.md#-accessing-aws)
- [IAM Users](../README.md#-iam-users)

---

## 📚 References

- AWS CLI User Guide  
  https://docs.aws.amazon.com/cli/latest/userguide/

- AWS CLI Configuration  
  https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html

- IAM Best Practices  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html