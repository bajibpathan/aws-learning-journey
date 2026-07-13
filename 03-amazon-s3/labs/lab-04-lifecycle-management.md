# 🧪 Lab 04 – Configure Amazon S3 Lifecycle Management

> Configure Amazon S3 Lifecycle Rules to automatically transition objects to lower-cost storage classes and expire objects based on predefined policies.

---

# 🎯 Objective

Learn how to automate object lifecycle management in Amazon S3 by creating Lifecycle Rules that transition objects between storage classes and delete objects after a specified retention period.

---

# 📋 Prerequisites

- AWS Account
- Existing Amazon S3 Bucket
- IAM User with Amazon S3 permissions
- Sample objects uploaded to the bucket

---

# ⚙️ Implementation Steps

### Step 1

Create or select an existing Amazon S3 bucket.

### Step 2

Upload one or more sample objects.

### Step 3

Navigate to:

**Bucket → Management → Lifecycle Rules**

### Step 4

Create a new Lifecycle Rule.

### Step 5

Choose the scope of the rule:

- Entire bucket
- Specific prefix
- Objects with specific tags

### Step 6

Configure Transition Actions.

Example:

- Transition to Standard-IA after 30 days.
- Transition to Glacier Instant Retrieval after 90 days.

### Step 7

Configure Expiration Actions.

Example:

- Permanently delete objects after 365 days.

### Step 8

Save the Lifecycle Rule.

---

# ✅ Validation

Verified:

- Lifecycle Rule created successfully.
- Rule scope applied to the intended objects.
- Transition actions configured correctly.
- Expiration actions configured correctly.
- Lifecycle Rule displayed under the bucket's **Management** tab.

---

# ⚠️ Important Observation

Lifecycle Rules are **not executed immediately**.

Amazon S3 evaluates Lifecycle Rules periodically, so transitions and expirations may take time to occur after the configured criteria are met.

---

# 💡 Lessons Learned

- Lifecycle Rules automate storage management.
- Transition Rules help reduce storage costs.
- Expiration Rules automatically remove obsolete data.
- Lifecycle Rules can manage current and previous object versions when Versioning is enabled.
- Lifecycle Rules work best when object access patterns are predictable.

---

# 🔒 Production Best Practices

- Define Lifecycle Rules based on business requirements and access patterns.
- Combine Lifecycle Rules with Versioning to automatically remove old object versions.
- Transition infrequently accessed data to lower-cost storage classes.
- Configure Expiration Rules for temporary files and obsolete data.
- Review Lifecycle Policies periodically to ensure they remain aligned with compliance and retention requirements.
- Use Intelligent-Tiering instead of Lifecycle Rules when object access patterns are unpredictable.

---

# 🎯 Result

Successfully configured Amazon S3 Lifecycle Rules to automate object transitions and expiration, reducing manual administration and helping optimize long-term storage costs.

---

# 📖 Related Concepts

- Amazon S3 Storage Classes
- Amazon S3 Versioning
- Amazon S3 Lifecycle Management
- Amazon S3 Intelligent-Tiering