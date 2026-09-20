# 🖼️ Amazon Machine Images (AMI)

> An Amazon Machine Image provides the information required to launch an Amazon EC2 instance. Organizations can also create custom AMIs to standardize operating systems, software, security configuration, and other server settings.

---

# 📖 Overview

When I launch an EC2 instance, I need a starting image that defines what the server should initially contain.

That is the purpose of an:

```text
Amazon Machine Image
        │
        ▼
       AMI
```

A simple way for me to think about it is:

```text
AMI
 │
 │ Launch
 ▼
EC2 Instance
```

The AMI acts as the template from which the EC2 instance is created.

An AMI can define characteristics such as:

```text
Operating System

CPU Architecture

Root Volume

Block Device Mapping

Preinstalled Software

Operating System Configuration
```

This makes AMIs useful not only for launching individual EC2 instances but also for creating standardized server configurations.

---

# 🧠 AMI vs EC2 Instance

An AMI is not a running server.

```text
AMI
 │
 │ Template
 ▼
EC2 Instance
 │
 │ Running Compute
 ▼
Application
```

The AMI provides the starting configuration.

The EC2 instance is the running compute resource created from it.

---

# 🏗️ Sources of AMIs

When launching EC2 instances, AMIs can come from several sources.

```text
AMI Sources
│
├── AWS-provided AMIs
│
├── AWS Marketplace AMIs
│
├── Shared / Public AMIs
│
└── Custom AMIs
```

---

# 🟠 AWS-Provided AMIs

AWS provides images for commonly used operating systems.

For example:

```text
Amazon Linux

Windows Server
```

These provide a trusted starting point for building EC2 instances.

---

# 🛒 AWS Marketplace AMIs

AWS Marketplace provides AMIs containing software from third-party vendors.

Instead of:

```text
Launch Server
     │
     ▼
Install Software
     │
     ▼
Configure Software
```

a vendor may provide:

```text
Vendor AMI
    │
    ▼
Launch EC2
    │
    ▼
Software Already Installed
```

Marketplace products may have software charges in addition to normal AWS infrastructure charges.

---

# 🌐 Shared and Public AMIs

AMI owners can share AMIs with other AWS accounts.

Public AMIs are available to all AWS accounts.

However:

> A public or shared AMI should not automatically be considered trusted.

AWS recommends performing appropriate due diligence when using AMIs created by third parties.

For enterprise environments, trusted providers and organizational controls should be used where appropriate.

---

# 🏢 Custom AMIs

One of the most useful concepts I learned is that organizations can create their own AMIs.

For example:

```text
Standard Amazon Linux
        │
        ▼
Launch EC2
        │
        ├── Patch OS
        ├── Install Apache
        ├── Install monitoring agent
        ├── Configure security
        ├── Install utilities
        └── Apply company configuration
        │
        ▼
Create Custom AMI
```

The resulting AMI becomes a reusable server template.

---

# 🥇 Golden Images

Organizations commonly create standardized server images sometimes called:

```text
Golden Image

or

Pre-baked AMI
```

For example:

```text
Corporate Linux AMI
│
├── Approved OS version
├── Security configuration
├── Monitoring agent
├── Standard packages
├── Logging configuration
└── Required utilities
```

Teams can then launch instances from this approved baseline rather than configuring every server manually.

---

# 🎯 Why Build Custom AMIs?

Without a standard image:

```text
Server A → Manual Configuration

Server B → Manual Configuration

Server C → Manual Configuration
```

This can introduce configuration differences.

With a custom AMI:

```text
             Corporate AMI
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Server A  Server B  Server C
```

All servers start from the same baseline.

This can improve:

* Consistency
* Deployment speed
* Standardization
* Security baseline enforcement
* Repeatability

---

# 🌎 AMIs Are Regional Resources

An AMI exists within an AWS Region.

For example:

```text
ca-central-1
     │
     └── Corporate AMI
```

I cannot directly use that Regional AMI as an AMI in another Region.

If I need it elsewhere:

```text
ca-central-1
     │
     │ Copy AMI
     ▼
us-east-1
     │
     └── Copied Corporate AMI
```

This is important for multi-Region architectures and disaster recovery planning.

---

# 💾 EBS-Backed AMIs

For an EBS-backed EC2 instance, the AMI is associated with EBS snapshots.

Consider:

```text
EC2 Instance
│
├── Root EBS Volume
│
└── Data EBS Volume
```

When creating an EBS-backed AMI, EC2 can create snapshots associated with the image.

Conceptually:

```text
EC2 Instance
│
├── Root EBS
│      │
│      ▼
│   Snapshot A
│
└── Data EBS
       │
       ▼
    Snapshot B

        │
        ▼

       AMI
```

The AMI contains block-device mapping information describing the storage configuration used when launching instances.

---

# 🧩 Block Device Mapping

Block-device mapping tells EC2 which storage devices should be available when an instance is launched.

Conceptually:

```text
AMI
│
├── Root Device
│      └── Snapshot A
│
└── Additional Device
       └── Snapshot B
```

When the AMI is launched:

```text
AMI
 │
 ▼
New EC2 Instance
│
├── New Root EBS Volume
│
└── New Data EBS Volume
```

The volumes are created based on the AMI's block-device mappings and associated snapshots.

---

# 🔄 Creating a Custom AMI

The basic process is:

```text
Launch EC2
    │
    ▼
Configure Server
    │
    ├── Patch OS
    ├── Install Software
    ├── Configure Services
    └── Apply Security Settings
    │
    ▼
Create Image
    │
    ▼
AMI
    │
    ▼
Launch New EC2 Instances
```

---

# 🔄 Reboot During AMI Creation

When creating an EBS-backed AMI, EC2 normally reboots the instance before creating the image.

Why?

```text
Application / OS
       │
       ▼
Buffered Data
       │
       ▼
Write to EBS
       │
       ▼
Snapshot
```

The reboot helps ensure buffered data is written before snapshots are created.

AWS allows AMI creation without rebooting, but this creates crash-consistent snapshots and AWS warns that filesystem integrity is not guaranteed in the same way.

For a normal lab, I would keep the default reboot behavior.

---

# 🔐 AMI Permissions

AMI launch permissions determine who can launch instances from an AMI.

The main categories are:

```text
AMI Permissions
│
├── Implicit
│     └── Owner
│
├── Explicit
│     ├── AWS Account
│     ├── AWS Organization
│     └── Organizational Unit
│
└── Public
      └── All AWS Accounts
```

A custom AMI should normally remain private unless there is a specific reason to share it.

---

# 🛡️ Block Public Access for AMIs

AWS also provides:

```text
Block Public Access for AMIs
```

This helps prevent AMIs from accidentally being made public.

For enterprise environments, preventing unintended public AMI sharing can be an important security control.

---

# 🔐 Encrypted AMIs and Sharing

Encryption introduces another consideration.

If an AMI is backed by encrypted snapshots and needs to be shared with other accounts or organizational units, the KMS configuration matters.

In particular, encrypted snapshots used for this type of sharing need an appropriate customer managed KMS key, and the receiving principals also need permission to use that key.

Therefore:

```text
AMI Permission
      +
Snapshot Permission
      +
KMS Permission
```

may all matter when sharing encrypted AMIs.

---

# 🔑 What is NOT Stored in the AMI?

An important distinction is that not every EC2 launch setting becomes part of the AMI.

For example, settings such
