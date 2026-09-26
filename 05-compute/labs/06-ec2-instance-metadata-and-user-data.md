# 🧪 Lab 06 EC2 Instance Metadata and User Data

> Explore IMDSv2 from an Amazon Linux EC2 instance and then use User Data to automatically install a web server and create a webpage using instance metadata.

---

# 🎯 Lab Objectives

By completing this lab, I should be able to:

* Explain what EC2 Instance Metadata is
* Locate the IMDS endpoint
* Explain why IMDSv2 uses tokens
* Request an IMDSv2 token
* Retrieve metadata values
* Explain the difference between IMDSv1 and IMDSv2
* Use EC2 User Data
* Bootstrap an Amazon Linux instance
* Install Apache automatically
* Query metadata from a User Data script
* Generate instance-specific web content
* Troubleshoot User Data execution

---

# 🏗️ Architecture

```text
                    Internet
                       │
                       ▼
                   HTTP :80
                       │
                       ▼
                 EC2 Web Server
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
          User Data           IMDSv2
              │                 │
              ▼                 ▼
       Install Apache      Instance Metadata
              │                 │
              │        ┌────────┼────────┐
              │        ▼        ▼        ▼
              │      Host     AMI ID   Instance ID
              │        │        │        │
              └────────┴────────┴────────┘
                       │
                       ▼
                  index.html
```

---

# 📋 Resources Required

```text
1 VPC

1 Public Subnet

1 Security Group

2 Small Amazon Linux EC2 Instances
```

Why two?

```text
Instance 1
   │
   ▼
Explore IMDSv2 manually


Instance 2
   │
   ▼
Test User Data + IMDSv2
```

I do **not** need to create an IMDSv1 instance for normal practice.

The course compares both versions for learning purposes, but my hands-on implementation will keep IMDSv2 required.

---

# Phase 1: Create the Security Group

Create:

```text
ec2-metadata-lab-sg
```

For the web-server portion:

```text
HTTP
TCP 80
```

can be allowed for temporary browser testing.

For management access, use the secure connection method available in my environment.

If direct SSH is used:

```text
SSH
TCP 22
Source: My IP
```

Do not unnecessarily expose SSH to:

```text
0.0.0.0/0
```

---

# Phase 2: Launch the Metadata Test Instance

Launch:

```text
Name:
imds-v2-lab
```

Use:

```text
Amazon Linux 2023
```

and a small instance type suitable for the lab.

Under:

```text
Advanced Details
```

configure:

```text
Metadata accessible:
Enabled

Metadata version:
V2 only (token required)
```

AWS exposes these settings when launching an EC2 instance and allows IMDSv2 to be required.

Launch the instance.

---

# Phase 3: Connect to the Instance

Connect using the appropriate management method.

Once connected:

```bash
whoami
```

Then:

```bash
hostname
```

This confirms that I am working inside the EC2 instance.

---

# Phase 4: Try Accessing Metadata Without a Token

Run:

```bash
curl -i \
  http://169.254.169.254/latest/meta-data/instance-id
```

Because the instance requires IMDSv2, a request without the required token should not return the instance metadata successfully.

This proves:

```text
IMDSv2 Required
       │
       ▼
No Token
       │
       ▼
Metadata Request Rejected
```

---

# Phase 5: Request an IMDSv2 Token

Now request a token:

```bash
TOKEN=$(curl -s -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

Check that a value exists:

```bash
echo ${#TOKEN}
```

I should avoid printing real tokens unnecessarily in production logs.

The token requested here can remain valid for up to:

```text
21,600 seconds
      =
6 hours
```

---

# Phase 6: List Metadata Categories

Run:

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/
```

I should see categories such as:

```text
ami-id

hostname

instance-id

instance-type

local-ipv4

mac

network/

placement/
```

---

# Phase 7: Retrieve Instance ID

Run:

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

Compare the result with the EC2 console.

They should match.

---

# Phase 8: Retrieve Instance Type

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-type
```

Expected example:

```text
t3.micro
```

The actual result depends on the instance I launched.

---

# Phase 9: Retrieve AMI ID

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/ami-id
```

Compare it with:

```text
EC2 Console
   │
   ▼
Instance Details
   │
   ▼
AMI ID
```

---

# Phase 10: Retrieve Private IP

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/local-ipv4
```

Compare this with the EC2 console.

---

# Phase 11: Retrieve Public IP

If the instance has a public IPv4 address:

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/public-ipv4
```

If no public IPv4 address exists, I should not assume the command is broken. The metadata item may simply not have a value for that instance.

---

# Phase 12: Retrieve Availability Zone

Run:

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone
```

Example:

```text
us-east-1a
```

Now I have dynamically discovered:

```text
Instance ID

Instance Type

AMI ID

Private IP

Public IP

Availability Zone
```

without using the AWS CLI or manually copying values from the console.

---

# ✅ Validation Checkpoint 1

I should now be able to explain:

```text
Where did the information come from?

169.254.169.254


Did I call the EC2 API?

No


Did IMDSv2 require a token?

Yes
```

---

# Phase 13: Launch the User Data Web Server

Now launch a second instance:

```text
Name:
userdata-web-lab
```

Use:

```text
Amazon Linux 2023
```

Attach:

```text
ec2-metadata-lab-sg
```

Keep:

```text
Metadata version:
V2 only
```

The lesson similarly launches a web server with IMDSv2 and uses User Data to retrieve metadata for the generated page.

---

# Phase 14: Add User Data

Under:

```text
Advanced Details
      │
      ▼
User Data
```

paste:

```bash
#!/bin/bash

dnf install -y httpd

systemctl enable httpd
systemctl start httpd

TOKEN=$(curl -s -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

INSTANCE_ID=$(curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

INSTANCE_TYPE=$(curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-type)

AMI_ID=$(curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/ami-id)

AZ=$(curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)

cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>EC2 Metadata Lab</title>
</head>
<body>
    <h1>EC2 Instance Metadata Lab</h1>

    <p><strong>Instance ID:</strong> $INSTANCE_ID</p>
    <p><strong>Instance Type:</strong> $INSTANCE_TYPE</p>
    <p><strong>AMI ID:</strong> $AMI_ID</p>
    <p><strong>Availability Zone:</strong> $AZ</p>
</body>
</html>
EOF

systemctl restart httpd
```

Launch the instance.

---

# 🧠 What Is the Script Doing?

The flow is:

```text
#!/bin/bash
     │
     ▼
Install Apache
     │
     ▼
Enable Apache
     │
     ▼
Start Apache
     │
     ▼
Request IMDSv2 Token
     │
     ▼
Retrieve Metadata
     │
     ├── Instance ID
     ├── Instance Type
     ├── AMI ID
     └── Availability Zone
     │
     ▼
Store Values in Variables
     │
     ▼
Generate index.html
     │
     ▼
Restart Apache
```

---

# Phase 15: Test the Website

Wait for:

```text
EC2 Status Checks
      │
      ▼
2/2 Passed
```

Then open:

```text
http://<PUBLIC-IP>
```

Expected page:

```text
EC2 Instance Metadata Lab

Instance ID: i-xxxxxxxx
Instance Type: t3.micro
AMI ID: ami-xxxxxxxx
Availability Zone: us-east-1a
```

Compare the values with the EC2 console.

They should match.

---

# 🎯 What Did I Just Prove?

The page itself was not manually configured after launch.

Instead:

```text
EC2 Launch
    │
    ▼
User Data
    │
    ▼
Install Apache
    │
    ▼
IMDSv2
    │
    ▼
Discover Instance Information
    │
    ▼
Generate Website
```

This is:

```text
Bootstrapping
       +
Dynamic Configuration
```

---

# Phase 16: Inspect User Data Execution

Connect to the web-server instance.

Check Apache:

```bash
sudo systemctl status httpd
```

Check the generated page:

```bash
cat /var/www/html/index.html
```

Then inspect cloud-init logs:

```bash
sudo less /var/log/cloud-init-output.log
```

This file is extremely useful when User Data does not behave as expected.

---

# 💥 Break/Fix Exercise 1: IMDSv2 Without a Token

Try:

```bash
curl -i \
  http://169.254.169.254/latest/meta-data/instance-id
```

Why does it fail?

Because:

```text
IMDSv2 Required
       +
No Token
       =
Rejected Request
```

Now repeat using the token.

---

# 💥 Break/Fix Exercise 2: Wrong Metadata Path

Try:

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/not-a-real-item
```

This demonstrates that:

```text
IMDS Working
```

does not automatically mean:

```text
Every Path Is Valid
```

When troubleshooting, verify the metadata category.

---

# 💥 Break/Fix Exercise 3: User Data Failure

Suppose the webpage does not load.

Troubleshoot in this order:

```text
Is EC2 Running?
      │
      ▼
Did 2/2 Checks Pass?
      │
      ▼
Is HTTP Allowed?
      │
      ▼
Is httpd Installed?
      │
      ▼
Is httpd Running?
      │
      ▼
Does index.html Exist?
      │
      ▼
Check cloud-init-output.log
```

Useful commands:

```bash
sudo systemctl status httpd
```

```bash
sudo ls -l /var/www/html/
```

```bash
sudo cat /var/www/html/index.html
```

```bash
sudo tail -100 /var/log/cloud-init-output.log
```

---

# 💥 Break/Fix Exercise 4: Disable Metadata Access

As an optional exercise, modify the instance metadata options so IMDS access is disabled.

Then try querying:

```text
169.254.169.254
```

The request should no longer work.

Before doing this on real workloads, remember that applications and agents can depend on IMDS.

Restore the setting when the exercise is complete.

---

# 🔍 Troubleshooting Reference

| Problem                | Check                                 |
| ---------------------- | ------------------------------------- |
| Metadata request fails | Token and IMDS settings               |
| `401 Unauthorized`     | Missing/invalid IMDSv2 token          |
| User Data did not work | `cloud-init-output.log`               |
| Apache not running     | `systemctl status httpd`              |
| Browser cannot connect | Security group TCP 80                 |
| Metadata value empty   | Verify that value exists for instance |
| Script syntax error    | cloud-init logs                       |
| Wrong instance details | Verify metadata path                  |

---

# ❓ Interview Questions

### Q1. What is the difference between Instance Metadata and User Data?

Instance Metadata describes the running instance.

User Data supplies configuration information or scripts to the instance.

### Q2. What endpoint did we use for IMDS?

```text
169.254.169.254
```

### Q3. Why did the first metadata request fail?

The instance required IMDSv2, but the request did not contain a token.

### Q4. How did we fix it?

We requested an IMDSv2 token and included it in subsequent metadata requests.

### Q5. Why did we use User Data?

To bootstrap the EC2 instance automatically during launch.

### Q6. Why did the webpage show different information for the instance?

The User Data script dynamically retrieved metadata from the instance on which it executed.

### Q7. Where would you troubleshoot a failed Linux User Data script?

A useful starting point is:

```text
/var/log/cloud-init-output.log
```

### Q8. Why shouldn't credentials be hardcoded in User Data?

User Data is not a secure secret-storage mechanism.

### Q9. How would this concept help Auto Scaling?

Every new instance could bootstrap itself automatically and retrieve its own instance-specific configuration.

### Q10. Why is IMDSv2 important in production?

It provides a token-based access model and additional defenses against unintended metadata access.

---

# 🧹 Cleanup

Terminate:

```text
imds-v2-lab

userdata-web-lab
```

Then remove:

```text
ec2-metadata-lab-sg
```

if it was created specifically for this lab.

Do not delete an existing VPC or subnet that was not created for this exercise.

---

# ✅ Completion Criteria

I consider this lab complete when I can:

* [ ] Explain Instance Metadata
* [ ] Explain User Data
* [ ] Explain IMDSv1 vs IMDSv2
* [ ] Identify `169.254.169.254`
* [ ] Request an IMDSv2 token
* [ ] Retrieve Instance ID
* [ ] Retrieve Instance Type
* [ ] Retrieve AMI ID
* [ ] Retrieve IP information
* [ ] Retrieve Availability Zone
* [ ] Explain token TTL
* [ ] Launch an instance with User Data
* [ ] Automatically install Apache
* [ ] Query metadata from User Data
* [ ] Generate a dynamic webpage
* [ ] Troubleshoot User Data using cloud-init logs
* [ ] Explain why secrets should not be stored in User Data
* [ ] Clean up the resources

---

# 💡 Key Takeaways

* Instance Metadata describes the EC2 instance.
* User Data helps configure the EC2 instance.
* IMDS is accessed locally from the instance.
* IMDSv2 requires a session token.
* IMDSv2 should be preferred over IMDSv1.
* User Data is useful for bootstrapping.
* User Data and IMDS can work together.
* Scripts can dynamically configure instances using metadata.
* cloud-init logs are important for troubleshooting Linux User Data.
* These concepts become especially useful with Launch Templates and Auto Scaling.

---

# 📚 Next Related Topics

* Launch Templates
* EC2 Auto Scaling
* Golden AMIs
* IAM Roles for EC2
* cloud-init
* Systems Manager
* Secrets Manager
* Infrastructure as Code
