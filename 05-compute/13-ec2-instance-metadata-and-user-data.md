# 🖥️ EC2 Instance Metadata and User Data

> EC2 Instance Metadata provides information about a running EC2 instance, while EC2 User Data provides information or scripts that can be supplied when the instance is launched.

---

# 📖 Overview

Two EC2 features that initially sound similar are:

```text
Instance Metadata

User Data
```

But they solve different problems.

The easiest way for me to remember them is:

```text
Instance Metadata
       │
       ▼
Information ABOUT the instance


User Data
       │
       ▼
Information or scripts PROVIDED TO the instance
```

For example:

```text
Need Instance ID?
      │
      ▼
Instance Metadata


Need to install Apache at launch?
      │
      ▼
User Data
```

---

# 1. EC2 Instance Metadata

Instance Metadata is information about a running EC2 instance that software on the instance can use to configure or manage itself.

Examples include:

```text
Instance ID

Instance Type

AMI ID

Hostname

Private IP Address

Public IP Address

MAC Address

Availability Zone

Security Groups

IAM-related information
```

AWS exposes metadata through the:

```text
Instance Metadata Service
          │
          ▼
         IMDS
```

The lesson demonstrates retrieving values such as the instance ID, public IP address, MAC address, and instance type.

---

# 🌐 Instance Metadata Endpoint

The IPv4 link-local address used by IMDS is:

```text
169.254.169.254
```

A metadata request is made from the EC2 instance itself.

Conceptually:

```text
EC2 Instance
     │
     │ HTTP request
     ▼
169.254.169.254
     │
     ▼
Instance Metadata
```

A typical metadata path looks like:

```text
/latest/meta-data/
```

Therefore:

```text
http://169.254.169.254/latest/meta-data/
```

represents the top-level metadata endpoint.

AWS also supports an IPv6 IMDS endpoint on supported Nitro instances and IPv6-enabled subnets.

---

# 🔎 Metadata Categories

The metadata endpoint contains different categories.

Examples include:

```text
ami-id

hostname

instance-id

instance-type

local-ipv4

mac

network/

placement/

public-ipv4

security-groups
```

Think of it like:

```text
/latest/meta-data/
       │
       ├── instance-id
       ├── instance-type
       ├── ami-id
       ├── hostname
       ├── local-ipv4
       ├── public-ipv4
       ├── mac
       └── network/
```

AWS maintains the current metadata categories.

---

# 🧠 Why Is Metadata Useful?

Suppose I build a generic AMI.

The same AMI could launch:

```text
EC2-A

EC2-B

EC2-C
```

Each instance has different information:

```text
Instance ID

IP Address

Hostname

Availability Zone
```

Instead of hardcoding these values into my application, the application can discover them dynamically.

```text
Application Starts
       │
       ▼
Query IMDS
       │
       ▼
Discover Instance Information
       │
       ▼
Configure Application
```

This makes automation much more flexible.

---

# 🔐 IMDSv1 vs IMDSv2

The Instance Metadata Service supports:

```text
IMDS
│
├── IMDSv1
│
└── IMDSv2
```

The major difference is how requests are authenticated.

---

# 🟠 IMDSv1

IMDSv1 uses a simple:

```text
Request
   │
   ▼
Response
```

model.

Conceptually:

```bash
curl http://169.254.169.254/latest/meta-data/instance-id
```

No session token is required.

That simplicity also provides less protection against certain application vulnerabilities and misconfigurations that could allow unintended access to metadata.

The course demonstrates IMDSv1 specifically so the difference from IMDSv2 can be observed.

---

# 🔴 Why IMDSv1 Can Be Risky

One important security concern is:

```text
Server-Side Request Forgery
          │
          ▼
         SSRF
```

Imagine a vulnerable application accepts a URL from an attacker:

```text
Attacker
   │
   ▼
Vulnerable Application
   │
   ▼
169.254.169.254
   │
   ▼
Instance Metadata
```

If an application can be manipulated into requesting metadata, sensitive information could potentially be exposed.

This is one reason IMDSv2 provides additional protections.

---

# 🟢 IMDSv2

IMDSv2 uses a:

```text
Session-Oriented
Token-Based
Request Model
```

Instead of directly requesting metadata:

```text
Request Token
      │
      ▼
Receive Token
      │
      ▼
Request Metadata + Token
      │
      ▼
Receive Metadata
```

AWS recommends using IMDSv2 and allows instances to be configured so that IMDSv2 is required and IMDSv1 requests fail.

---

# 🔑 IMDSv2 Token

The first step is obtaining a token.

Example:

```bash
TOKEN=$(curl -s -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

The token TTL can range from:

```text
1 second

to

21,600 seconds
```

which is:

```text
6 hours
```

The token can then be reused during its valid lifetime.

---

# 🔎 Query Metadata with IMDSv2

Once I have the token:

```bash
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

Architecture:

```text
EC2
 │
 │ PUT
 ▼
IMDS
 │
 ▼
Token
 │
 │ GET + Token
 ▼
IMDS
 │
 ▼
Instance ID
```

---

# 🆚 IMDSv1 vs IMDSv2

| Feature                                     | IMDSv1           | IMDSv2               |
| ------------------------------------------- | ---------------- | -------------------- |
| Request model                               | Request/response | Session-oriented     |
| Token required                              | No               | Yes                  |
| Protection against certain SSRF-style paths | Lower            | Stronger             |
| Recommended for new workloads               | No               | Yes                  |
| Can be disabled                             | Yes              | N/A when v2 required |

A simple mental model:

```text
IMDSv1
Request → Metadata


IMDSv2
Token → Request + Token → Metadata
```

---

# ⚙️ EC2 Metadata Options

EC2 allows control over metadata behavior.

Important options include:

```text
Metadata Accessible

Metadata Version

HTTP PUT Response Hop Limit

Instance Tags in Metadata
```

An instance can be configured as:

```text
IMDSv1 or IMDSv2
Token Optional
```

or:

```text
IMDSv2 Only
Token Required
```

AWS also allows metadata options to be influenced at the account, AMI, and instance levels.

For modern workloads, a strong default is:

```text
IMDS Enabled
      +
IMDSv2 Required
```

unless there is a tested compatibility reason otherwise.

---

# 🏷️ Instance Tags Through Metadata

EC2 can optionally expose instance tags through IMDS.

This capability is:

```text
Disabled

or

Enabled
```

through metadata options.

If enabled, applications running on the instance can retrieve permitted instance tags through IMDS.

This can be useful for configuration such as:

```text
Environment = PROD

Application = Payments

Role = WebServer
```

However, it should be enabled intentionally rather than assumed to be available.

---

# 2. EC2 User Data

Instance Metadata answers:

```text
What information does this instance have about itself?
```

User Data answers a different question:

```text
What should happen when this instance is launched?
```

User Data can be used to supply configuration information or scripts to an EC2 instance.

The course uses User Data as a bootstrap mechanism to install and configure software automatically.

---

# 🚀 Bootstrapping

Using a script during instance launch is commonly called:

```text
Bootstrapping
```

Instead of:

```text
Launch EC2
    │
    ▼
Connect Manually
    │
    ▼
Install Apache
    │
    ▼
Start Apache
    │
    ▼
Create Website
```

I can automate it:

```text
Launch EC2
    │
    ▼
User Data
    │
    ├── Install Apache
    ├── Start Apache
    └── Create Website
```

---

# 🐧 Linux User Data

Linux instances commonly use shell scripts.

For example:

```bash
#!/bin/bash

dnf install -y httpd

systemctl enable httpd
systemctl start httpd

echo "Hello from EC2" > /var/www/html/index.html
```

The first line:

```bash
#!/bin/bash
```

tells the operating system which interpreter should process the script.

---

# 🪟 Windows User Data

Windows EC2 instances can use PowerShell.

Conceptually:

```text
Linux EC2
   │
   ▼
Shell / Bash


Windows EC2
   │
   ▼
PowerShell
```

This makes User Data useful across different EC2 operating systems.

---

# 🔢 User Data and Base64

EC2 APIs work with User Data as:

```text
Base64-encoded data
```

When I enter plain-text User Data through tools that perform the encoding for me, I normally do not manually Base64-encode it.

The important concept is:

```text
My Script
    │
    ▼
Base64 Representation
    │
    ▼
EC2 User Data
```

I should not confuse:

```text
Base64 Encoding
```

with:

```text
Encryption
```

Base64 is **not a security mechanism**.

---

# ⚠️ Do Not Put Secrets in User Data

Because User Data is not inherently a secure secret store, I should avoid embedding things such as:

```text
Passwords

API Keys

Database Credentials

Private Tokens
```

For secrets, I should use purpose-built services such as:

```text
AWS Secrets Manager

AWS Systems Manager Parameter Store
```

with appropriate IAM controls.

---

# 🔄 When Does User Data Run?

The normal Linux EC2 behavior is that User Data shell scripts and cloud-init directives run during the initial launch cycle.

So my mental model should be:

```text
Launch Instance
      │
      ▼
First Boot
      │
      ▼
User Data
      │
      ▼
Bootstrap Instance
```

The lesson correctly treats User Data primarily as a bootstrap mechanism.

It is possible to configure behavior that runs commands again on subsequent boots, but that requires additional configuration and should not be assumed from ordinary User Data behavior.

---

# 🆚 Golden AMI vs User Data

These two techniques complement each other.

## Golden AMI

```text
AMI
│
├── Operating System
├── Common Software
├── Agents
└── Base Configuration
```

Then User Data can perform instance-specific configuration:

```text
Launch from Golden AMI
        │
        ▼
User Data
        │
        ├── Environment Configuration
        ├── Instance-Specific Settings
        └── Application Startup
```

This gives me:

```text
Golden AMI
    +
User Data
    =
Repeatable EC2 Deployment
```

---

# 🔗 Combining Metadata and User Data

The most interesting part of this lesson is combining both features.

```text
EC2 Launch
    │
    ▼
User Data Executes
    │
    ▼
Request IMDSv2 Token
    │
    ▼
Query Instance Metadata
    │
    ├── Hostname
    ├── Public IP
    └── AMI ID
    │
    ▼
Generate Webpage
```

The lesson uses exactly this approach to create a webpage dynamically from instance information.

This demonstrates an important automation principle:

> User Data can bootstrap an instance, while Instance Metadata can provide instance-specific values to that bootstrap process.

---

# 🏗️ Example Architecture

```text
                 Launch EC2
                     │
                     ▼
                  User Data
                     │
              ┌──────┴──────┐
              │             │
              ▼             ▼
        Install Apache    Query IMDSv2
                              │
                              ▼
                     Instance Metadata
                              │
                  ┌───────────┼──────────┐
                  ▼           ▼          ▼
              Hostname    Public IP    AMI ID
                  │           │          │
                  └───────────┼──────────┘
                              ▼
                         index.html
                              │
                              ▼
                         Web Browser
```

---

# ⚠️ Common Mistakes

### Mistake 1: Confusing Metadata and User Data

```text
Metadata
   =
Information ABOUT instance


User Data
   =
Information/scripts PROVIDED TO instance
```

---

### Mistake 2: Using IMDSv1 for New Workloads

Prefer IMDSv2 and require tokens unless compatibility testing identifies a reason not to.

---

### Mistake 3: Hardcoding Instance Information

Instead of hardcoding:

```text
AMI ID

Instance ID

Hostname
```

an application can retrieve appropriate values dynamically from metadata.

---

### Mistake 4: Assuming User Data Runs Every Reboot

Ordinary User Data scripts should be thought of primarily as first-launch bootstrap scripts.

---

### Mistake 5: Thinking Base64 Means Encrypted

```text
Base64 != Encryption
```

Do not use User Data as a secret store.

---

### Mistake 6: Disabling IMDS Without Testing

Some applications, agents, and AWS SDK functionality can depend on IMDS.

Test dependencies before disabling metadata access entirely.

---

# ✅ Best Practices

* Prefer IMDSv2.
* Require IMDSv2 where application compatibility allows.
* Do not expose sensitive metadata unnecessarily.
* Disable IMDS entirely when workloads do not require it and this has been tested.
* Use IAM roles rather than hardcoded AWS credentials.
* Never treat Base64 as encryption.
* Avoid storing secrets in User Data.
* Keep User Data scripts small and repeatable.
* Log bootstrap activity for troubleshooting.
* Test User Data before using it in production.
* Consider Golden AMIs for large, stable software installations.
* Use User Data for dynamic instance-specific configuration.
* Use IMDS when software needs to discover properties of the instance on which it is running.

---

# ❓ Interview Questions

### Q1. What is EC2 Instance Metadata?

Information about a running EC2 instance that applications and scripts on the instance can use for configuration and management.

### Q2. What address is associated with the IPv4 Instance Metadata Service?

```text
169.254.169.254
```

### Q3. What is IMDSv2?

The session-oriented version of EC2 Instance Metadata Service that requires a token for metadata requests.

### Q4. What is the main difference between IMDSv1 and IMDSv2?

IMDSv1 does not require a session token. IMDSv2 requires obtaining a token and including it with metadata requests.

### Q5. How long can an IMDSv2 token remain valid?

From one second to six hours.

### Q6. Why is IMDSv2 preferred?

It provides additional protections against several paths that could otherwise expose instance metadata.

### Q7. What is EC2 User Data?

Data supplied to an EC2 instance at launch, commonly used for bootstrap scripts and initial configuration.

### Q8. What is bootstrapping?

Automatically configuring an instance during its initial launch.

### Q9. Can Linux and Windows both use User Data?

Yes. Linux commonly uses shell/cloud-init mechanisms, while Windows can use PowerShell-based initialization.

### Q10. Is Base64 encryption?

No. It is encoding, not encryption.

### Q11. Should passwords be stored in User Data?

No. Use an appropriate secrets-management solution.

### Q12. Does User Data normally execute every reboot?

No. The common default behavior is initial-launch execution. Repeated execution requires additional configuration.

### Q13. Can User Data query Instance Metadata?

Yes. A bootstrap script can use IMDS to obtain instance-specific values.

### Q14. Can an instance be configured to reject IMDSv1?

Yes. Configure the instance so that IMDSv2 tokens are required.

### Q15. Can instance tags be retrieved through metadata?

Yes, when access to tags through instance metadata has explicitly been enabled.

---

# 💡 Key Takeaways

* Instance Metadata provides information about the EC2 instance.
* User Data provides configuration information or scripts to the EC2 instance.
* IMDS uses the link-local IPv4 address `169.254.169.254`.
* IMDSv1 uses simple request/response access.
* IMDSv2 uses session tokens.
* IMDSv2 tokens can have a TTL from one second to six hours.
* IMDSv2 should be preferred for modern workloads.
* User Data is commonly used for EC2 bootstrapping.
* Linux User Data commonly uses shell scripts.
* Windows can use PowerShell.
* User Data is Base64 encoded at the API level, but Base64 is not encryption.
* Secrets should not be placed in User Data.
* User Data and Instance Metadata can be combined for dynamic configuration.
* Golden AMIs and User Data complement each other.
* Metadata options can be controlled at account, AMI, and instance levels.

---

# 📚 Related Topics

* Amazon EC2
* EC2 User Data
* EC2 Instance Metadata Service
* IMDSv2
* IAM Roles for EC2
* AMIs
* Golden AMIs
* cloud-init
* Bash
* PowerShell
* AWS Secrets Manager
* Systems Manager Parameter Store
* EC2 Auto Scaling
* Launch Templates
