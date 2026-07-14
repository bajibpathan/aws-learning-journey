# 🧪 Lab 06 – Configure CORS for an Amazon S3 Bucket

> Configure Cross-Origin Resource Sharing (CORS) on an Amazon S3 bucket to allow a web application hosted on another origin to access bucket resources securely.

---

# 🎯 Objective

Learn how to configure CORS on an Amazon S3 bucket and validate cross-origin requests from a web application hosted on a different origin.

---

# 📋 Prerequisites

- AWS Account
- IAM User with Amazon S3 permissions
- Existing Amazon S3 bucket
- Sample objects uploaded to the bucket
- Static website or web application hosted on a different origin

---

# ⚙️ Implementation Steps

### Step 1

Create or select an existing Amazon S3 bucket.

### Step 2

Upload one or more sample files (images, HTML, JSON, etc.).

### Step 3

Navigate to:

**Bucket → Permissions → Cross-origin resource sharing (CORS)**

### Step 4

Add a CORS configuration.

Example:

```json
[
  {
    "AllowedHeaders": [
      "*"
    ],
    "AllowedMethods": [
      "GET"
    ],
    "AllowedOrigins": [
      "http://example.com"
    ],
    "ExposeHeaders": []
  }
]
```

> Replace `http://example.com` with your application's origin.

### Step 5

Save the CORS configuration.

### Step 6

Open the web application hosted on the allowed origin.

### Step 7

Access the S3 object from the browser.

### Step 8

Verify the request using the browser's Developer Tools (Network tab).

---

# ✅ Validation

Verified:

- CORS configuration saved successfully.
- Browser successfully accessed the S3 object.
- Response included the **Access-Control-Allow-Origin** header.
- Cross-origin request completed successfully.

---

# ⚠️ Issue Faced

Initially, the browser blocked the request with a **CORS policy** error.

### Root Cause

The origin of the web application was not included in the S3 bucket's CORS configuration.

### Resolution

- Added the application's origin to the **AllowedOrigins** section.
- Saved the updated CORS configuration.
- Refreshed the browser and verified the request.

---

# 💡 Lessons Learned

- CORS is enforced by web browsers, not by Amazon S3.
- Bucket permissions and CORS configuration are both required for browser access.
- Only trusted origins should be allowed.
- CORS does not replace IAM Policies or Bucket Policies.
- Browser Developer Tools are useful for troubleshooting CORS issues.

---

# 🔒 Production Best Practices

- Configure CORS only when cross-origin access is required.
- Allow only trusted origins instead of using the wildcard (`*`) whenever possible.
- Restrict allowed HTTP methods to the minimum required.
- Follow the Principle of Least Privilege.
- Periodically review and remove unused CORS rules.

---

# 🎯 Result

Successfully configured Cross-Origin Resource Sharing (CORS) for an Amazon S3 bucket, allowing a trusted web application hosted on a different origin to securely access bucket resources while maintaining browser security controls.

---

# 📖 Related Concepts

- Amazon S3
- Cross-Origin Resource Sharing (CORS)
- Bucket Policies
- Static Website Hosting
- Amazon CloudFront