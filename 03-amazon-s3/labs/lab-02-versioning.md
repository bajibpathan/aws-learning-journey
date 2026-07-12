# 🧪 Lab 02 – Enable Amazon S3 Versioning

> Enable Versioning on an Amazon S3 bucket and validate object version history and Delete Marker behavior.

---

# 🎯 Objective

Enable Amazon S3 Versioning and validate how Amazon S3 manages multiple object versions and accidental deletions.

---

# 📋 Prerequisites

- Existing Amazon S3 bucket
- AWS Console access
- Appropriate IAM permissions

---

# 🏗 Architecture

```
Amazon S3 Bucket

↓

Versioning Enabled

↓

Object Versions

↓

Delete Marker
```

---

# ⚙️ Implementation Steps

- Reviewed the existing bucket.
- Enabled Versioning.
- Uploaded multiple versions of the same object.
- Verified object versions.
- Deleted the object.
- Verified Delete Marker creation.
- Removed the Delete Marker.

---

# ✅ Validation

Verified:

- Versioning enabled successfully.
- Multiple object versions existed.
- Delete Marker was created.
- Removing the Delete Marker restored the object.

---

# ⚠️ Issue Encountered

## Observation

Deleting the object did not permanently remove it.

### Explanation

Because Versioning was enabled, Amazon S3 created a Delete Marker instead of deleting previous versions.

### Resolution

Enabled **Show Versions** and removed the Delete Marker.

---

# 💡 Lessons Learned

- Versioning protects against accidental deletion.
- Previous versions remain available.
- Delete Markers hide objects rather than permanently deleting them.
- Lifecycle Rules help control storage costs.

---

# 🔒 Production Best Practices

- Enable Versioning for production buckets.
- Configure Lifecycle Rules.
- Monitor storage growth.
- Consider Cross-Region Replication for disaster recovery.

---

# 🎯 Result

Successfully enabled Versioning and validated object version history and Delete Marker functionality.

---

# 📖 Related Concepts

- Amazon S3 Versioning
- Lifecycle Rules