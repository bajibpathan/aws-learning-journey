# 🧪 Lab 01 – Create an Amazon S3 Bucket

> Create an Amazon S3 bucket, securely grant access to an IAM User using a Bucket Policy, and validate access through the AWS CLI.

---

# 🎯 Objective

Create an Amazon S3 bucket and provide least-privilege access to an IAM User using a Bucket Policy.

Validate bucket access using the AWS CLI.

---

# 📋 Prerequisites

- AWS Account
- IAM User
- AWS CLI installed
- AWS CLI configured
- Appropriate IAM permissions

---

# 🏗 Architecture

```
IAM User
     │
AWS CLI
     │
Bucket Policy
     │
Amazon S3 Bucket
```

---

# ⚙️ Implementation Steps

- Created an Amazon S3 bucket.
- Configured a Bucket Policy granting access to a specific IAM User.
- Configured the AWS CLI using a named profile.
- Uploaded an object using the AWS CLI.
- Verified object upload.
- Listed bucket contents.

---

# ✅ Validation

Verified:

- Bucket was successfully created.
- IAM User authenticated successfully.
- Object upload completed successfully.
- Bucket Policy granted only the required permissions.
- Uploaded object was visible in the bucket.

---

# 💻 Commands Used

```bash
aws configure --profile simon-dev

aws s3 cp sample.txt s3://bucket-name --profile simon-dev

aws s3 ls s3://bucket-name --profile simon-dev
```

---

# ⚠️ Issue Encountered

## Problem

Attempted to list all buckets:

```bash
aws s3 ls --profile simon-dev
```

Received an **AccessDenied** error.

### Root Cause

The IAM User had access to the specific bucket but did not have the `s3:ListAllMyBuckets` permission.

### Resolution

Validated access using bucket-specific commands instead of granting unnecessary permissions.

### Lesson

Grant only the permissions required by the application.

---

# 💡 Lessons Learned

- Bucket Policies provide resource-level access control.
- IAM Policies and Bucket Policies work together.
- AWS CLI uses IAM credentials for authentication.
- Least Privilege improves security.

---

# 🔒 Production Best Practices

- Keep Block Public Access enabled.
- Grant only required permissions.
- Use IAM Roles instead of long-term credentials where possible.
- Apply meaningful bucket naming standards.

---

# 🎯 Result

Successfully created an Amazon S3 bucket, configured secure access using a Bucket Policy, and validated access using the AWS CLI.

---

# 📖 Related Concepts

- Amazon S3
- Bucket Policies
- IAM Policies