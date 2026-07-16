# 🧪 Lab 08 – Configure Amazon S3 Encryption

> Learn how to configure and verify different Amazon S3 server-side encryption options to protect objects stored in an S3 bucket.

---

# 🎯 Objective

Learn how to encrypt Amazon S3 objects using different Server-Side Encryption (SSE) options and understand how Amazon S3 protects data at rest.

During this lab, you will:

- Configure default bucket encryption
- Upload encrypted objects
- Verify object encryption
- Compare SSE-S3 and SSE-KMS

---

# 📋 Prerequisites

- AWS Account
- IAM User with S3 permissions
- Existing Amazon S3 bucket (or permission to create one)
- AWS KMS permissions (for SSE-KMS)

---

# ⚙️ Implementation Steps

## Part 1 – Configure Default SSE-S3 Encryption

### Step 1

Navigate to:

**Amazon S3 → Buckets**

### Step 2

Create a new bucket (or select an existing bucket).

Example:

```
my-encrypted-bucket
```

### Step 3

Open the bucket.

Navigate to:

**Properties → Default Encryption**

### Step 4

Choose:

- Server-side encryption
- Amazon S3 managed keys (SSE-S3)

Click **Save Changes**.

---

## Part 2 – Upload an Object

### Step 5

Navigate to:

**Bucket → Objects**

Upload a sample file.

Example:

```
sample.txt
```

Since default encryption is enabled, Amazon S3 automatically encrypts the object using SSE-S3.

---

## Part 3 – Verify Encryption

### Step 6

Select the uploaded object.

Navigate to:

**Properties**

Under **Server-side Encryption**, verify:

```
Amazon S3 Managed Keys (SSE-S3)
```

Encryption Status:

```
Encrypted
```

---

## Part 4 – Configure SSE-KMS

### Step 7

Navigate to:

**Bucket → Properties → Default Encryption**

Select:

- AWS Key Management Service (AWS KMS)

Choose:

- AWS Managed Key (`aws/s3`)
  **OR**
- Customer Managed KMS Key

Save the configuration.

---

## Part 5 – Upload Another Object

### Step 8

Upload another file.

Example:

```
customer-data.csv
```

Amazon S3 encrypts the object using the selected KMS key.

---

## Part 6 – Verify SSE-KMS

### Step 9

Open the uploaded object.

Verify:

- Encryption Type
- KMS Key ID
- Encryption Status

Example:

```
AWS KMS Key
Key ID: aws/s3
```

---

## Part 7 – (Optional) Upload Using REST API

Upload an object using SSE-S3:

```
x-amz-server-side-encryption: AES256
```

Upload an object using SSE-KMS:

```
x-amz-server-side-encryption: aws:kms
```

---

# ✅ Validation

Verified:

- Default bucket encryption configured successfully.
- Objects uploaded with SSE-S3 were automatically encrypted.
- SSE-KMS encryption worked successfully.
- KMS Key ID was visible in the object properties.
- Uploaded objects were encrypted at rest.

---

# ⚠️ Issue Faced

### Issue 1

Unable to upload an object using SSE-KMS.

### Root Cause

IAM user did not have permission to use the AWS KMS key.

### Resolution

Granted the required AWS KMS permissions:

- `kms:Encrypt`
- `kms:Decrypt`
- `kms:GenerateDataKey`

Successfully uploaded the object after updating the IAM policy.

---

### Issue 2

Object was encrypted using SSE-S3 instead of SSE-KMS.

### Root Cause

Bucket default encryption was still configured to use Amazon S3 Managed Keys.

### Resolution

Updated the bucket's default encryption to AWS KMS and uploaded the object again.

---

# 💡 Lessons Learned

- Amazon S3 automatically encrypts all newly uploaded objects.
- Encryption is applied at the object level.
- Bucket default encryption determines the default encryption method for new uploads.
- SSE-S3 is simple and fully managed by AWS.
- SSE-KMS provides additional security through AWS KMS, IAM policies, and CloudTrail auditing.
- KMS permissions are required in addition to Amazon S3 permissions when using SSE-KMS.

---

# 🔒 Production Best Practices

- Always enable default bucket encryption.
- Use HTTPS for all uploads and downloads.
- Use **SSE-KMS** for production workloads requiring compliance, auditing, and fine-grained access control.
- Follow the Principle of Least Privilege when granting KMS permissions.
- Enable AWS CloudTrail to audit KMS key usage.
- Rotate customer-managed KMS keys according to organizational security policies.
- Use Customer Managed KMS Keys (CMKs) when applications require strict control over encryption.

---

# 🎯 Result

Successfully configured Amazon S3 Server-Side Encryption using both SSE-S3 and SSE-KMS, uploaded encrypted objects, verified their encryption settings, and understood how Amazon S3 protects data at rest using different encryption options.

---

# 📖 Related Concepts

- Amazon S3
- Amazon S3 Encryption
- Server-Side Encryption (SSE)
- Client-Side Encryption (CSE)
- AWS Key Management Service (AWS KMS)
- AWS IAM
- AWS CloudTrail
- Data Encryption at Rest
- Data Encryption in Transit