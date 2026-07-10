# 🧪 Lab 03 - Create an IAM Policy and Grant Amazon S3 Access

## 📖 Lab Information

| Property | Value |
|----------|-------|
| **Category** | AWS Identity & Access Management |
| **Level** | 🟢 Beginner |
| **Estimated Time** | 20–25 Minutes |
| **AWS Services** | IAM, Amazon S3 |

---

## 🎯 Objective

Create a custom IAM policy to grant an IAM user read-only access to a specific Amazon S3 bucket while following the Principle of Least Privilege.

---

## 📋 Prerequisites

- AWS Account
- Administrator permissions
- Two Amazon S3 buckets
- IAM User

---

## ⚙️ Implementation Steps

1. Create two Amazon S3 buckets.
2. Upload one or more objects to one of the buckets.
3. Create an IAM User.
4. Sign in as the IAM User and attempt to access the S3 bucket.
5. Observe that access is denied due to the default implicit deny behavior.
6. Create a custom IAM Policy granting:
   - `s3:ListBucket`
   - `s3:GetObject`
7. Attach the policy to the IAM User.
8. Sign in again as the IAM User.
9. Verify that the user can access only the authorized bucket and objects.

---

## ✅ Validation

Verify the following:

- The IAM User receives an **AccessDenied** error before the policy is attached.
- After attaching the policy, the IAM User can:
  - View the authorized S3 bucket.
  - List objects in the bucket.
  - Download objects from the bucket.
- The IAM User cannot access unauthorized S3 buckets.

---

## 📄 IAM Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BucketActions",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-first-aws-ritual-roast-bucket",
        "arn:aws:s3:::my-first-aws-ritual-roast-bucket/*"
      ]
    },
    {
      "Sid": "ListBucketsInAccount",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```

---

## 💻 Commands Used *(Optional)*

```bash
aws s3 ls

aws s3 ls s3://my-first-aws-ritual-roast-bucket

aws s3 cp s3://my-first-aws-ritual-roast-bucket/sample.txt .
```

---

## 🎯 Result

Successfully implemented fine-grained access control by granting an IAM User read-only access to a specific Amazon S3 bucket using a custom IAM Policy.

---

## 💡 Lessons Learned

- IAM follows an **implicit deny** model by default.
- Permissions must be explicitly granted using IAM Policies.
- Custom IAM Policies provide fine-grained access control.
- The Principle of Least Privilege helps reduce unnecessary permissions.
- Separate ARNs are required for the bucket itself and the objects within the bucket.

---

## ❓ Frequently Asked Question

### Why are two ARNs required in the IAM Policy?

Amazon S3 treats the bucket and the objects inside the bucket as separate resources.

- Bucket ARN:
  ```text
  arn:aws:s3:::my-first-aws-ritual-roast-bucket
  ```

  Used for bucket-level actions such as:

  - `s3:ListBucket`

- Object ARN:
  ```text
  arn:aws:s3:::my-first-aws-ritual-roast-bucket/*
  ```

  Used for object-level actions such as:

  - `s3:GetObject`
  - `s3:PutObject`
  - `s3:DeleteObject`

Both ARNs are required because bucket-level and object-level permissions are evaluated independently.

---

## 🔗 Related Documentation

- [IAM Overview](../README.md)
- [IAM Policies](../README.md#-iam-policies)


---

## 📚 References

- AWS IAM Policies  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html

- Amazon S3 Policy Actions  
  https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-policy-actions.html

- IAM JSON Policy Reference  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html