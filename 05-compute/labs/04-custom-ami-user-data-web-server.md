# 🧪 Lab 04: Build and Deploy a Custom AMI with User Data

> Build a reusable Amazon Linux AMI, launch a new EC2 instance from it, use an IAM role to securely access S3, and use EC2 User Data to deploy website content automatically.

---

# 🎯 Lab Objective

This lab brings several EC2 concepts together:

```text
EC2
 +
EBS
 +
AMI
 +
S3
 +
IAM
 +
User Data
```

Instead of manually configuring every server, I will build a reusable image.

Then I will launch another EC2 instance from that image and dynamically configure it using User Data.

---

# 🏗️ Architecture

```text
                  PHASE 1
                     │
                     ▼
             Amazon Linux AMI
                     │
                     ▼
                Builder EC2
                     │
              Install Apache
                     │
                     ▼
               Custom AMI
                     │
                     │
       ┌─────────────┴─────────────┐
       │                           │
       │                        PHASE 2
       │                           │
       │                           ▼
       │                     Launch EC2
       │                           │
       │                ┌──────────┴──────────┐
       │                │                     │
       │                ▼                     ▼
       │            IAM Role              User Data
       │                │                     │
       │                ▼                     │
       │               S3 ◄───────────────────┘
       │                │
       │                ▼
       │          Website Files
       │                │
       │                ▼
       └──────────► Apache Web Server
```

---

# 📋 Prerequisites

I need:

* An AWS account
* A VPC
* A subnet suitable for this lab
* Permission to work with EC2
* Permission to create IAM roles/policies
* Permission to create an S3 bucket
* Basic Linux command familiarity

For this learning lab, direct browser access to the instance can be used if networking is configured appropriately.

For production architectures, internet-facing traffic would normally be handled through architecture such as a load balancer rather than directly exposing application EC2 instances.

---

# 🧱 Phase 1: Build the Source EC2 Instance

Launch an EC2 instance.

Example:

```text
Name:
ami-builder

AMI:
Amazon Linux 2023

Instance Type:
Small lab-appropriate instance

VPC:
Existing Lab VPC

Subnet:
Lab Subnet
```

Use EC2 Instance Connect or Session Manager where available rather than unnecessarily exposing SSH.

---

# 🔧 Phase 2: Configure the Server

Connect to:

```text
ami-builder
```

Update packages:

```bash
sudo dnf update -y
```

Install Apache:

```bash
sudo dnf install -y httpd
```

Start Apache:

```bash
sudo systemctl start httpd
```

Enable Apache at boot:

```bash
sudo systemctl enable httpd
```

Verify:

```bash
sudo systemctl status httpd
```

Expected:

```text
active (running)
```

---

# ✍️ Create a Simple Test Page

Create:

```bash
echo '<h1>Custom AMI Test</h1>' | sudo tee /var/www/html/index.html
```

Verify locally:

```bash
curl http://localhost
```

Expected content:

```text
Custom AMI Test
```

---

# 🧠 Why Install Apache Before Creating the AMI?

Apache represents:

```text
Stable Server Configuration
```

that every web server should have.

Instead of:

```text
Launch Server
    ↓
Install Apache

Launch Server
    ↓
Install Apache

Launch Server
    ↓
Install Apache
```

we want:

```text
Custom AMI
│
└── Apache Installed
       │
       ├── Server 1
       ├── Server 2
       └── Server 3
```

---

# 📸 Phase 3: Create the Custom AMI

From the EC2 console:

```text
Instances
    │
    ▼
ami-builder
    │
    ▼
Actions
    │
    ▼
Image and Templates
    │
    ▼
Create Image
```

Example:

```text
Image Name:
corporate-web-ami-v1

Description:
Amazon Linux web server baseline with Apache
```

Keep the default reboot behavior for this lab.

Create the image.

---

# ⏳ Phase 4: Monitor AMI Creation

Navigate to:

```text
EC2
 │
 ▼
AMIs
```

Initially:

```text
pending
```

Wait until:

```text
available
```

---

# 🔍 Phase 5: Inspect the AMI

Record:

```text
AMI ID

AMI Name

Architecture

Root Device Type

Creation Date
```

Then inspect the associated snapshot.

Conceptually:

```text
ami-builder
     │
     ▼
Root EBS
     │
     ▼
Snapshot
     │
     ▼
Custom AMI
```

This connects the previous EBS lessons with AMIs.

---

# 🪣 Phase 6: Create an S3 Bucket

Now I want launch-time configuration to be separate from the AMI.

Create an S3 bucket.

Example:

```text
baji-ami-lab-<unique-suffix>
```

Bucket names must be globally unique.

Keep:

```text
Block Public Access
```

enabled.

The EC2 instance does not need the bucket to be public because it will access S3 using IAM.

---

# 🌐 Phase 7: Create Website Content

Create a local `index.html` file containing:

```html
<!DOCTYPE html>
<html>
<head>
    <title>AMI Lab</title>
</head>
<body>
    <h1>Deployed from a Custom AMI</h1>
    <p>Website content downloaded securely from Amazon S3.</p>
</body>
</html>
```

Upload it to the S3 bucket.

Architecture:

```text
S3 Bucket
   │
   └── index.html
```

---

# 🔐 Phase 8: Create a Least-Privilege IAM Policy

The EC2 instance only needs to retrieve website objects.

For this simple lab, create a policy scoped to the specific bucket objects.

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadWebsiteFiles",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

Replace:

```text
YOUR-BUCKET-NAME
```

with the actual bucket name.

---

# 🧠 Why Not Use AmazonS3FullAccess?

The server only needs:

```text
Read Website Files
```

It does not need:

```text
Delete Objects

Create Buckets

Modify Bucket Policies

Delete Buckets
```

Therefore:

```text
Required Permission
       │
       ▼
s3:GetObject
```

is enough for this specific deployment pattern.

This follows:

```text
Principle of Least Privilege
```

---

# 👤 Phase 9: Create an IAM Role for EC2

Create a role.

Trusted entity:

```text
AWS Service
    │
    ▼
EC2
```

Attach the custom S3 policy.

Example role name:

```text
ami-lab-web-role
```

Conceptually:

```text
EC2
 │
 ▼
IAM Role
 │
 ▼
IAM Policy
 │
 ▼
S3 Bucket
```

No long-term AWS access keys need to be stored on the EC2 instance.

---

# 🚀 Phase 10: Launch EC2 from the Custom AMI

Navigate to:

```text
EC2
 │
 ▼
AMIs
 │
 ▼
corporate-web-ami-v1
 │
 ▼
Launch Instance from AMI
```

Example:

```text
Name:
ami-web-server

AMI:
corporate-web-ami-v1

Instance Type:
Small lab-appropriate instance

VPC:
Lab VPC

Subnet:
Lab Subnet
```

---

# 🔐 Phase 11: Attach the IAM Role

Under:

```text
Advanced Details
```

select:

```text
IAM Instance Profile:
ami-lab-web-role
```

Now:

```text
EC2
 │
 │ Temporary Credentials
 ▼
IAM Role
 │
 ▼
S3
```

---

# 📜 Phase 12: Add User Data

Add:

```bash
#!/bin/bash

aws s3 cp s3://YOUR-BUCKET-NAME/index.html /var/www/html/index.html

systemctl enable httpd
systemctl restart httpd
```

Replace the bucket name.

---

# 🧠 What is User Data Doing?

The AMI already contains:

```text
Amazon Linux
      +
Apache
```

User Data adds:

```text
Environment-Specific
Website Content
```

So:

```text
Custom AMI
│
├── OS
└── Apache
      │
      ▼
Launch EC2
      │
      ▼
User Data
      │
      ▼
Download index.html
from S3
      │
      ▼
Running Website
```

This is the important architectural lesson from the lab.

---

# ⏱️ User Data Execution

For standard Linux EC2 behavior, User Data runs during the first boot by default.

Therefore:

```text
Launch Instance
      │
      ▼
cloud-init
      │
      ▼
User Data
      │
      ▼
Download Website
```

The instance may show:

```text
running
```

before the User Data script has completely finished.

Allow a little time before testing.

---

# 🔍 Phase 13: Validate User Data

Connect to the instance.

Check:

```bash
ls -l /var/www/html/
```

Verify:

```bash
cat /var/www/html/index.html
```

Then:

```bash
sudo systemctl status httpd
```

Test locally:

```bash
curl http://localhost
```

---

# 🐛 Troubleshooting User Data

If the website file is missing, inspect cloud-init logs.

Useful logs include:

```bash
sudo cat /var/log/cloud-init-output.log
```

Also check:

```bash
sudo cat /var/log/cloud-init.log
```

Look for:

```text
S3 AccessDenied

Bucket Not Found

Incorrect Bucket Name

AWS CLI Error

Script Syntax Error
```

---

# 🌐 Phase 14: Test the Website

If the lab intentionally allows HTTP access to this EC2 instance, configure the security group for:

```text
HTTP
TCP 80
```

For a short learning exercise, the source can be configured according to the test requirement.

Do not reuse a bastion security group simply because one already exists.

Create a security group whose purpose is clear, for example:

```text
ami-lab-web-sg
```

Then browse to:

```text
http://<instance-address>
```

Expected:

```text
Deployed from a Custom AMI

Website content downloaded securely from Amazon S3.
```

---

# 🔎 Phase 15: Verify IAM Access

From the EC2 instance:

```bash
aws sts get-caller-identity
```

Then:

```bash
aws s3 cp s3://YOUR-BUCKET-NAME/index.html /tmp/test-index.html
```

Verify:

```bash
cat /tmp/test-index.html
```

This confirms:

```text
EC2
 │
 ▼
IAM Role
 │
 ▼
S3
 │
 ▼
GetObject
```

---

# 🧪 Break/Fix Exercise 1: Remove S3 Permission

Detach the S3 policy from the IAM role.

Attempt:

```bash
aws s3 cp s3://YOUR-BUCKET-NAME/index.html /tmp/index.html
```

Observe the failure.

Restore the policy.

### Lesson

```text
Network Connectivity
        ≠
Authorization
```

The instance may be able to reach S3 but still be denied access by IAM.

---

# 🧪 Break/Fix Exercise 2: Wrong Bucket Name

Change the command to reference a nonexistent bucket.

Observe the failure.

### Lesson

User Data automation can fail because of configuration errors even when IAM is correct.

---

# 🧪 Break/Fix Exercise 3: Stop Apache

Run:

```bash
sudo systemctl stop httpd
```

Test:

```bash
curl http://localhost
```

Then troubleshoot:

```bash
sudo systemctl status httpd
```

Restore:

```bash
sudo systemctl start httpd
```

### Lesson

A healthy EC2 instance does not necessarily mean the application running on it is healthy.

---

# 🧪 Break/Fix Exercise 4: Security Group

Temporarily remove the HTTP inbound rule.

Test from the browser.

Then restore it.

### Lesson

```text
Application Running
       +
Correct Network Access
       =
Reachable Application
```

Application health and network reachability are separate concerns.

---

# 🔄 Phase 16: Understand the Full Deployment

We now have:

```text
                 Custom AMI
                 │
                 ├── Amazon Linux
                 └── Apache
                        │
                        ▼
                   Launch EC2
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
         IAM Role               User Data
            │                       │
            └─────────┐   ┌─────────┘
                      ▼   ▼
                        S3
                         │
                         ▼
                    index.html
                         │
                         ▼
                  /var/www/html
                         │
                         ▼
                       Apache
                         │
                         ▼
                      Website
```

---

# 🧠 AMI vs User Data Decision

After this lab, the distinction is clearer:

| Requirement                  | AMI | User Data |
| ---------------------------- | --: | --------: |
| Operating system baseline    |   ✅ |           |
| Apache installation          |   ✅ |  Possible |
| Security agent               |   ✅ |  Possible |
| Standard utilities           |   ✅ |  Possible |
| Website version              |     |         ✅ |
| Environment configuration    |     |         ✅ |
| Dynamic launch configuration |     |         ✅ |

A useful mental model is:

```text
Slow-Changing Baseline
        │
        ▼
       AMI


Launch-Specific Configuration
        │
        ▼
     User Data
```

---

# 🏢 Production Evolution

This lab performs the process manually so I can understand it.

A more mature architecture might evolve toward:

```text
EC2 Image Builder
       │
       ▼
Automated Golden AMI
       │
       ▼
Launch Template
       │
       ▼
Auto Scaling Group
       │
       ▼
Application Load Balancer
```

and potentially:

```text
CI/CD
 │
 ▼
Application Artifact
 │
 ▼
Deployment Automation
```

The manual lab is therefore teaching the underlying concepts that later automation builds upon.

---

# 💰 Cost Awareness

This lab can create billable resources including:

```text
EC2 Instances

EBS Volumes

EBS Snapshots associated with AMI

S3 Storage

Public IPv4 Address
```

Do not rely on historical course Free Tier assumptions.

Check current AWS pricing and account eligibility.

---

# 🧹 Cleanup

Terminate:

```text
ami-web-server

ami-builder
```

Delete the S3 objects.

Delete the S3 bucket.

Delete:

```text
ami-lab-web-role
```

Delete the custom IAM policy.

Delete the lab security group when it is no longer referenced.

---

# ⚠️ AMI Cleanup Requires an Extra Step

An AMI is not removed simply because the source EC2 instance is terminated.

If the AMI is no longer needed:

```text
AMI
 │
 ▼
Deregister AMI
```

Then review the associated EBS snapshots.

Conceptually:

```text
Deregister AMI
      │
      ▼
Review Associated
EBS Snapshots
      │
      ▼
Delete if No Longer Needed
```

Do not delete snapshots that are still needed by other images or recovery workflows.

---

# ✅ Lab Completion Criteria

I consider the lab complete when I can do these without blindly following the instructions:

```text
☐ Launch an Amazon Linux EC2 instance

☐ Install and configure Apache

☐ Explain why Apache belongs in the AMI for this lab

☐ Create a custom AMI

☐ Explain the relationship between AMI and EBS snapshots

☐ Explain why an AMI is Regional

☐ Create a private S3 bucket

☐ Upload website content

☐ Create least-privilege S3 permissions

☐ Create an IAM role for EC2

☐ Explain why access keys are not needed on EC2

☐ Launch an EC2 instance from my custom AMI

☐ Attach the IAM instance profile

☐ Configure User Data

☐ Download content from S3 automatically

☐ Verify the User Data execution

☐ Troubleshoot User Data using cloud-init logs

☐ Explain AMI vs User Data

☐ Troubleshoot an IAM failure

☐ Troubleshoot an application failure

☐ Troubleshoot a security-group failure

☐ Clean up all resources
```

---

# ❓ Interview Questions

### Q1. Why create a custom AMI?

To provide a reusable and standardized server baseline.

---

### Q2. Why use User Data if we already have a custom AMI?

The AMI can contain stable configuration while User Data applies dynamic or environment-specific configuration at launch.

---

### Q3. Why use an IAM role instead of AWS access keys?

The IAM role provides temporary credentials to the EC2 instance, avoiding hard-coded long-term credentials.

---

### Q4. Does the S3 bucket need to be public for EC2 to download files?

No.

The bucket can remain private while the EC2 instance accesses it using IAM authorization.

---

### Q5. When does Linux EC2 User Data normally run?

By default, Linux User Data scripts and cloud-init directives run during the initial launch boot cycle.

---

### Q6. Where can User Data execution be troubleshot?

On Amazon Linux, a useful starting point is:

```text
/var/log/cloud-init-output.log
```

along with other cloud-init logs.

---

### Q7. What happens if the IAM role lacks `s3:GetObject`?

The EC2 instance will not be authorized to retrieve the S3 object even if network connectivity exists.

---

### Q8. Is an IAM role part of the AMI?

No.

The IAM instance profile is associated with the EC2 instance.

---

### Q9. Is a security group stored inside an AMI?

No.

The security group is selected for the launched EC2 instance/network interface.

---

### Q10. What production services could build on this pattern?

Examples include:

```text
EC2 Image Builder

Launch Templates

Auto Scaling Groups

Application Load Balancer

Systems Manager

CI/CD
```

---

# 💡 Key Takeaways

* A custom AMI creates a reusable EC2 baseline.
* EBS-backed AMIs are associated with EBS snapshots and block-device mappings.
* AMIs are Regional resources.
* Golden AMIs help standardize server configuration.
* IAM roles allow EC2 workloads to access AWS services without storing long-term access keys.
* S3 buckets do not need to be public for EC2 workloads to retrieve objects.
* Least privilege should be used when granting S3 permissions.
* User Data provides launch-time automation.
* Linux User Data normally runs only during the initial launch by default.
* AMIs are useful for stable configuration.
* User Data is useful for dynamic configuration.
* Security groups, IAM roles, subnet selection, and instance types are separate from the AMI.
* A successful EC2 launch does not guarantee successful User Data execution.
* Cloud-init logs are important when troubleshooting EC2 initialization.
* The combination of AMI + IAM + User Data is a foundation for more advanced automated EC2 deployment patterns.

---

# 📚 Related Topics

* Amazon EC2
* Amazon Machine Images
* Amazon EBS Snapshots
* EC2 User Data
* cloud-init
* IAM Roles for EC2
* IAM Instance Profiles
* Amazon S3
* EC2 Image Builder
* Launch Templates
* Auto Scaling Groups
* Application Load Balancer
* AWS Systems Manager
* Immutable Infrastructure

---

# 📖 References

* AWS Documentation: Amazon Machine Images
* AWS Documentation: Create an Amazon EBS-backed AMI
* AWS Documentation: EC2 User Data
* AWS Documentation: IAM Roles for Amazon EC2
* AWS Documentation: Amazon S3 Identity-Based Policies
* AWS Documentation: EC2 Image Builder
