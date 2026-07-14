# 🧪 Lab 05 – Host a Static Website using Amazon S3

> Configure an Amazon S3 bucket to host a static website and make it accessible through the S3 Website Endpoint.

---

# 🎯 Objective

Learn how to host a static website using Amazon S3 by enabling Static Website Hosting, configuring the required bucket permissions, and validating website accessibility.

---

# 📋 Prerequisites

- AWS Account
- IAM User with Amazon S3 permissions
- Sample website files
  - `index.html`
  - `error.html`

---

# ⚙️ Implementation Steps

### Step 1

Create a new Amazon S3 bucket.

### Step 2

Upload the website files.

Example:

- `index.html`
- `error.html`

### Step 3

Navigate to:

**Bucket → Properties → Static Website Hosting**

### Step 4

Enable **Static Website Hosting**.

Configure:

- Index document: `index.html`
- Error document: `error.html`

### Step 5

Update the Bucket Policy to allow public read access to the website objects.

### Step 6

Verify that **Block Public Access** settings allow public website access if required.

### Step 7

Copy the **S3 Website Endpoint**.

### Step 8

Open the Website Endpoint in a browser and verify that the website loads successfully.

---

# ✅ Validation

Verified:

- Static Website Hosting enabled successfully.
- Website endpoint generated.
- `index.html` loaded successfully.
- `error.html` displayed for invalid pages.
- Website accessible through the browser.

---

# ⚠️ Issue Faced

Initially, the website returned an **Access Denied (403 Forbidden)** error.

**Root Cause**

The bucket objects were not publicly readable because the Bucket Policy did not grant public read access.

**Resolution**

- Updated the Bucket Policy to allow read access (`s3:GetObject`).
- Verified the Block Public Access settings.
- Refreshed the Website Endpoint.

The website loaded successfully after applying the correct permissions.

---

# 💡 Lessons Learned

- Static Website Hosting must be enabled before the website endpoint becomes available.
- The website requires an **Index Document**.
- Public read access must be granted through a Bucket Policy.
- Amazon S3 can host only static content and cannot execute server-side code.
- Production environments commonly place Amazon CloudFront in front of Amazon S3 to provide HTTPS, caching, and improved performance.

---

# 🔒 Production Best Practices

- Follow the Principle of Least Privilege when configuring Bucket Policies.
- Grant only **read-only** access to website objects.
- Use Amazon CloudFront for HTTPS, caching, and global content delivery.
- Enable Versioning to protect website files.
- Use Route 53 with a custom domain for production websites.
- Monitor bucket access using AWS CloudTrail and Amazon CloudWatch.

---

# 🎯 Result

Successfully hosted a static website on Amazon S3 by enabling Static Website Hosting, uploading website files, configuring the Bucket Policy, and validating access through the S3 Website Endpoint.

---

# 📖 Related Concepts

- Amazon S3
- Amazon S3 Static Website Hosting
- Bucket Policies
- Amazon CloudFront
- Amazon Route 53