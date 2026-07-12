# 🧪 Lab 03 – Configure Amazon S3 Replication

> Configure Amazon S3 Replication to automatically copy objects between buckets.

---

# 🎯 Objective

Configure Same-Region or Cross-Region Replication between two Amazon S3 buckets.

---

# 📋 Prerequisites

- Two Amazon S3 buckets
- Versioning enabled on both buckets
- IAM permissions
- Destination bucket created

---

# 🏗 Architecture

```
Source Bucket

↓

Versioning

↓

Replication Rule

↓

IAM Role

↓

Destination Bucket
```

---

# ⚙️ Implementation Steps

- Enabled Versioning on both buckets.
- Created the IAM Replication Role.
- Configured the Replication Rule.
- Selected destination bucket.
- Uploaded a new object.
- Verified replication.

---

# ✅ Validation

Verified:

- Replication rule created successfully.
- Object uploaded to the source bucket.
- Object automatically appeared in the destination bucket.
- Replication status showed successful completion.

---

# ⚠️ Issue Encountered

## Common Observation

Previously uploaded objects were not replicated.

### Explanation

Replication only applies to new object versions.

### Resolution

Upload a new object or use Amazon S3 Batch Replication for existing objects.

---

# 💡 Lessons Learned

- Replication requires Versioning.
- Replication is asynchronous.
- Existing objects require Batch Replication.
- IAM Roles are required for replication.

---

# 🔒 Production Best Practices

- Replicate only business-critical data.
- Follow Least Privilege.
- Monitor replication using CloudWatch.
- Test disaster recovery regularly.

---

# 🎯 Result

Successfully configured Amazon S3 Replication and verified automatic object replication between buckets.

---

# 📖 Related Concepts

- Amazon S3 Replication
- Versioning
- IAM Roles