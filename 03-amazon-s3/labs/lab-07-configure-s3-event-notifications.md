# 🧪 Lab 07 – Configure Amazon S3 Event Notifications

> Configure Amazon S3 Event Notifications to automatically trigger an AWS Lambda function whenever an object is uploaded to an S3 bucket.

---

# 🎯 Objective

Learn how to configure Amazon S3 Event Notifications to build an event-driven workflow by automatically invoking an AWS Lambda function when objects are uploaded to an S3 bucket.

---

# 📋 Prerequisites

- AWS Account
- IAM User with required permissions
- Existing Amazon S3 bucket
- AWS Lambda function
- Sample object for testing

---

# ⚙️ Implementation Steps

### Step 1

Create or select an existing Amazon S3 bucket.

### Step 2

Create an AWS Lambda function.

Example:

- Runtime: Python 3.x
- Function Name: `S3ObjectNotification`

### Step 3

Grant Amazon S3 permission to invoke the Lambda function.

Verify that the Lambda function contains a resource-based policy allowing:

- `lambda:InvokeFunction`
- Principal: `s3.amazonaws.com`

### Step 4

Navigate to:

**Amazon S3 Bucket → Properties → Event Notifications**

### Step 5

Create a new Event Notification.

Configure:

- Event Name
- Event Type:
  - Object Created (All)
- Destination:
  - AWS Lambda
- Lambda Function:
  - `S3ObjectNotification`

### Step 6

Save the Event Notification.

### Step 7

Upload a test object to the bucket.

Example:

```
sample-image.jpg
```

### Step 8

Navigate to Amazon CloudWatch Logs.

Verify that the Lambda function was invoked successfully.

---

# ✅ Validation

Verified:

- Event Notification created successfully.
- Lambda function invoked automatically.
- CloudWatch Logs captured the invocation.
- Uploaded object triggered the expected event.

---

# ⚠️ Issue Faced

Initially, Amazon S3 could not invoke the Lambda function.

### Root Cause

The Lambda function did not contain the required resource-based policy allowing Amazon S3 to invoke it.

### Resolution

- Added the required Lambda permission.
- Verified the resource-based policy.
- Uploaded another object.
- Successfully validated the event notification.

---

# 💡 Lessons Learned

- Amazon S3 Event Notifications enable event-driven architectures.
- Event Notifications are processed asynchronously.
- Resource-based policies are required on the destination service.
- CloudWatch Logs are useful for validating Lambda execution.
- Event Notifications eliminate the need for applications to continuously poll Amazon S3.

---

# 🔒 Production Best Practices

- Configure notifications only for the required event types.
- Follow the Principle of Least Privilege when granting permissions.
- Prevent recursive workflows by avoiding Lambda functions that continuously write back to the same bucket.
- Use Amazon EventBridge for advanced event routing and filtering.
- Monitor Lambda execution using Amazon CloudWatch.

---

# 🎯 Result

Successfully configured Amazon S3 Event Notifications to automatically invoke an AWS Lambda function whenever an object was uploaded to the S3 bucket, demonstrating a simple event-driven architecture.

---

# 📖 Related Concepts

- Amazon S3
- Amazon S3 Event Notifications
- AWS Lambda
- Amazon EventBridge
- Amazon SNS
- Amazon SQS
- Amazon CloudWatch