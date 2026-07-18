# 🔒 Amazon S3 Object Lock

> Learn how Amazon S3 Object Lock protects object versions from accidental or malicious deletion by enforcing **Write Once, Read Many (WORM)** protection for a specified retention period or until a legal hold is removed.

---

# 📖 Overview

Amazon S3 Object Lock is a data protection feature that prevents object versions from being permanently deleted or overwritten.

It is commonly used to meet regulatory, legal, and compliance requirements where data must remain immutable for a specified period.

Object Lock uses the **Write Once, Read Many (WORM)** model, ensuring that protected object versions cannot be modified or permanently deleted while the protection is in effect.

To use Object Lock, **S3 Versioning must be enabled**.

---

# 🎯 Learning Objectives

After completing this topic, you should understand:

- What Amazon S3 Object Lock is
- Why Object Lock is used
- Write Once, Read Many (WORM)
- Retention Period
- Governance Mode
- Compliance Mode
- Legal Hold
- Best practices
- Interview concepts

---

# 🔒 What is Amazon S3 Object Lock?

Amazon S3 Object Lock is a feature that protects **individual object versions** from being permanently deleted or overwritten.

It uses the **Write Once, Read Many (WORM)** model, allowing an object version to be written once and read many times without modification.

Object Lock helps organizations:

- Meet regulatory requirements
- Protect critical backups
- Preserve business records
- Prevent accidental deletion
- Protect against malicious deletion

Object Lock can apply:

- A **Retention Period**
- A **Legal Hold**
- Or both

---

# 🏗 How Object Lock Works

```text
Upload Object
        │
        ▼
Version 1 Created
        │
        ▼
Apply Object Lock
        │
 ┌──────┴─────────┐
 │                │
 ▼                ▼
Retention      Legal Hold
        │
        ▼
Object Version Protected
(No Permanent Delete / Overwrite)
```

Object Lock protects **specific object versions**.

If another object with the same name is uploaded, Amazon S3 creates a **new version**, while the protected version remains locked.

---

# 📝 Prerequisites

Before enabling Object Lock:

- Versioning must be enabled.
- Object Lock must be enabled on the bucket.
- Once Object Lock is enabled for a bucket, it **cannot be disabled**.

---

# ✍️ Write Once, Read Many (WORM)

WORM stands for:

> **Write Once, Read Many**

It means:

- Data can be written once.
- Data can be read multiple times.
- Protected object versions cannot be modified.
- Protected object versions cannot be permanently deleted until protection expires.

This model is commonly required by industries such as:

- Banking
- Healthcare
- Government
- Financial Services
- Insurance

---

# ⏳ Retention Period

A retention period protects an object version until a specified **Retain Until Date**.

Example:

```text
Retain Until:
31-Dec-2030
```

During the retention period:

- Object versions cannot be permanently deleted.
- Object versions cannot be overwritten.
- The retention period protects only that object version.

Amazon S3 provides two retention modes:

- Governance Mode
- Compliance Mode

---

# 🛡 Governance Mode

Governance Mode provides strong protection while still allowing authorized administrators to bypass the protection when necessary.

Characteristics:

- Protects object versions from deletion.
- Protects object versions from modification.
- Authorized users can bypass the protection.
- Requires:
  - `s3:BypassGovernanceRetention`
  - Explicit bypass request

Best for:

- Internal governance
- Corporate policies
- Backup protection
- Development and testing

---

# 🔐 Compliance Mode

Compliance Mode provides the highest level of protection.

Characteristics:

- No user can permanently delete the protected object version.
- No user can shorten the retention period.
- Even the AWS account root user cannot bypass the protection.
- The retention period can only be extended.

Best for:

- Financial records
- Healthcare data
- Government records
- Regulatory compliance
- SEC Rule 17a-4
- FINRA compliance

> **Important:** Use Compliance Mode carefully because the protection cannot be bypassed until the retention period expires.

---

# ⚖ Governance Mode vs Compliance Mode

| Feature | Governance Mode | Compliance Mode |
|----------|-----------------|-----------------|
| Prevents deletion | ✅ | ✅ |
| Prevents overwrite | ✅ | ✅ |
| Authorized user can bypass | ✅ | ❌ |
| Root user can bypass | ✅ (with required permissions) | ❌ |
| Retention period can be shortened | ✅ (authorized users only) | ❌ |
| Recommended for | Internal governance | Regulatory compliance |

---

# ⚖ Legal Hold

A Legal Hold protects an object version **indefinitely**.

Unlike a retention period:

- No expiration date.
- Remains active until an authorized user removes it.
- Does not depend on a retention period.

Common use cases:

- Legal proceedings
- Litigation
- Compliance audits
- Internal investigations
- Regulatory investigations

---

# 🔄 Retention Period vs Legal Hold

| Feature | Retention Period | Legal Hold |
|----------|-----------------|------------|
| Expiration | Fixed date | No expiration |
| Can expire automatically | ✅ | ❌ |
| Removed manually | Depends on mode | ✅ |
| Used for | Records retention | Legal preservation |

---

# ⭐ Key Characteristics

- Protects individual object versions.
- Uses Write Once, Read Many (WORM).
- Requires S3 Versioning.
- Supports Governance Mode.
- Supports Compliance Mode.
- Supports Legal Hold.
- Helps prevent accidental and malicious deletion.
- Commonly used for regulatory compliance.

---

# 💼 Common Use Cases

### Governance Mode

- Internal backup retention
- Corporate governance
- Protection against accidental deletion

### Compliance Mode

- Financial records
- Medical records
- Government archives
- SEC / FINRA compliance
- Long-term regulatory retention

### Legal Hold

- Legal proceedings
- Compliance investigations
- Corporate audits
- Preservation of evidence

---

# ✅ Benefits

- Prevents accidental deletion.
- Protects against malicious deletion.
- Supports regulatory compliance.
- Preserves important records.
- Protects backup data.
- Supports immutable storage.

---

# ⚠ Important Considerations

- Object Lock protects **object versions**, not the entire bucket.
- Versioning must be enabled.
- Object Lock must be enabled on the bucket.
- Object Lock cannot be disabled after it is enabled for a bucket.
- Uploading an object with the same name creates a new version.
- A protected version remains locked even if newer versions are uploaded.
- Storage charges continue while protected object versions are retained.
- IAM permissions still control who can read the protected object.

---

# 🔒 Best Practices

- Enable Object Lock only for buckets requiring immutable storage.
- Carefully review retention periods before using Compliance Mode.
- Use Governance Mode during testing or where administrative overrides may be required.
- Apply Compliance Mode only for strict regulatory requirements.
- Follow the Principle of Least Privilege.
- Restrict permissions such as:
  - `s3:PutObjectRetention`
  - `s3:GetObjectRetention`
  - `s3:PutObjectLegalHold`
  - `s3:GetObjectLegalHold`
  - `s3:BypassGovernanceRetention`
- Enable AWS CloudTrail to audit Object Lock changes.
- Apply Lifecycle rules where appropriate, while considering retention requirements.
- Monitor long-term storage costs for retained object versions.

---

# ❓ Frequently Asked Questions

### Q1. Does Object Lock require Versioning?

**Answer**

Yes.

Amazon S3 Versioning must be enabled before Object Lock can protect object versions.

---

### Q2. Can Object Lock be disabled after it is enabled?

**Answer**

No.

Once Object Lock is enabled for a bucket, it cannot be disabled.

---

### Q3. Can I upload another object with the same name?

**Answer**

Yes.

Amazon S3 creates a **new object version**, while the protected version remains locked.

---

### Q4. What is the difference between Governance Mode and Compliance Mode?

**Answer**

Governance Mode allows authorized users with special permissions to bypass retention.

Compliance Mode cannot be bypassed by any user, including the AWS account root user.

---

### Q5. Does a Legal Hold expire automatically?

**Answer**

No.

A Legal Hold remains active until an authorized user removes it.

---

### Q6. Can an object have both a Retention Period and a Legal Hold?

**Answer**

Yes.

An object version can have both protections applied simultaneously.

---

# 💡 Key Takeaways

- Amazon S3 Object Lock protects object versions using the **Write Once, Read Many (WORM)** model.
- Versioning must be enabled before Object Lock can be used.
- Object Lock provides two protection methods:
  - Retention Period
  - Legal Hold
- Retention Period supports:
  - Governance Mode
  - Compliance Mode
- Governance Mode allows authorized administrative bypass.
- Compliance Mode provides the highest level of protection and cannot be bypassed.
- Legal Hold remains active until explicitly removed.
- Object Lock is widely used for compliance, legal, and backup retention requirements.

---


# 🔗 Related Topics

- Amazon S3
- Amazon S3 Versioning
- Amazon S3 Lifecycle Rules
- Amazon S3 Replication
- AWS IAM
- AWS CloudTrail
- Write Once, Read Many (WORM)

---

# 📖 References

- AWS Documentation – Amazon S3 Object Lock

  https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html

- AWS Documentation – Managing Amazon S3 Object Lock

  https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock-managing.html

- AWS Documentation – Object Lock Retention Modes

  https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock-overview.html