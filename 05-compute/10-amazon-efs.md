# 📁 Amazon Elastic File System (Amazon EFS)

> Amazon EFS provides elastic, shared file storage that can be mounted concurrently by multiple supported compute resources using the Network File System (NFS) protocol.

---

# 📖 Overview

After learning about Amazon EBS, the main thing that became clear to me is that **block storage and shared file storage solve different problems**.

With EBS, the normal architecture looks like:

```text
EC2 Instance
     │
     ▼
EBS Volume
```

EBS provides block storage and is an excellent choice for operating systems, databases, and application data that need block-level access.

But what if several Linux servers need to work with the **same files**?

For example:

```text
Web Server A ──┐
Web Server B ──┼──► Shared Website Files
Web Server C ──┘
```

This is where:

```text
Amazon Elastic File System
           │
           ▼
          EFS
```

becomes useful.

---

# 🎯 Why Amazon EFS?

Imagine an application running on several EC2 instances.

```text
EC2-A

EC2-B

EC2-C
```

Each server could have its own EBS volume:

```text
EC2-A ──► EBS-A

EC2-B ──► EBS-B

EC2-C ──► EBS-C
```

But now each server has its own filesystem and its own copy of the data.

If the application requires shared files, I need a different storage model.

With EFS:

```text
EC2-A ──┐
        │
EC2-B ──┼──► Amazon EFS
        │
EC2-C ──┘
```

All three instances can access the same shared filesystem concurrently.

---

# 🧱 Block Storage vs File Storage

This distinction is fundamental.

## Amazon EBS

```text
Application
     │
     ▼
File System
     │
     ▼
Block Device
     │
     ▼
EBS
```

The operating system sees EBS as a block device.

Examples:

```text
/dev/xvda
/dev/nvme1n1
```

I then create a filesystem such as:

```text
XFS

EXT4
```

---

## Amazon EFS

With EFS, the filesystem itself is provided as a managed network service.

```text
Application
     │
     ▼
Mounted Directory
     │
     ▼
NFS
     │
     ▼
Amazon EFS
```

For example, Linux might mount EFS at:

```text
/mnt/efs
```

Applications then work with normal files and directories.

---

# 🆚 EBS vs EFS

| Feature               | Amazon EBS                         | Amazon EFS                        |
| --------------------- | ---------------------------------- | --------------------------------- |
| Storage model         | Block                              | File                              |
| Scope                 | Availability Zone                  | Regional or One Zone              |
| Common access pattern | Attached block device              | Shared network filesystem         |
| Protocol              | Block storage                      | NFS                               |
| Capacity              | Volume capacity configured         | Automatically grows/shrinks       |
| Multi-server sharing  | Specialized Multi-Attach scenarios | Designed for shared access        |
| Linux support         | Yes                                | Yes                               |
| Windows EC2 storage   | Yes                                | EFS Windows clients not supported |
| Typical use           | OS, databases, block storage       | Shared files                      |

A simple mental model:

```text
Need a Hard Drive?
       │
       ▼
      EBS


Need a Shared Linux Filesystem?
       │
       ▼
      EFS
```

---

# 🌎 EFS File System Types

Amazon EFS provides two file-system types:

```text
Amazon EFS
│
├── Regional
│
└── One Zone
```

The architecture and availability characteristics are different.

---

# 🟢 Regional EFS

A Regional EFS file system stores data redundantly across multiple Availability Zones within an AWS Region.

Conceptually:

```text
              AWS Region
┌──────────────────────────────────┐
│                                  │
│          Amazon EFS              │
│       Regional File System       │
│              │                   │
│     ┌────────┼────────┐          │
│     │        │        │          │
│     ▼        ▼        ▼          │
│   AZ-A     AZ-B     AZ-C         │
│                                  │
└──────────────────────────────────┘
```

This makes Regional EFS suitable when applications run across multiple Availability Zones and require shared file storage.

---

# 🆚 EBS Scope vs EFS Scope

This helped clarify the architecture for me.

## EBS

```text
Region
│
├── AZ-A
│    │
│    ├── EC2
│    └── EBS
│
└── AZ-B
     │
     ├── EC2
     └── Different EBS
```

An EBS volume belongs to a specific Availability Zone.

---

## Regional EFS

```text
             Region
               │
               ▼
         Regional EFS
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
      AZ-A           AZ-B
        │             │
   Mount Target  Mount Target
        │             │
        ▼             ▼
      EC2-A          EC2-B
```

The same EFS filesystem can be accessed from workloads across Availability Zones.

---

# 🔌 EFS Mount Targets

EC2 instances do not directly attach EFS like an EBS block device.

Instead, EFS uses:

```text
Mount Targets
```

A mount target provides network access to the EFS filesystem.

For a Regional filesystem:

```text
             Amazon EFS
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
      AZ-A              AZ-B
        │                 │
   Mount Target      Mount Target
        │                 │
        ▼                 ▼
      EC2-A              EC2-B
```

---

# 🌐 Mount Targets and Subnets

A mount target is created in a subnet.

For example:

```text
AZ-A
│
├── Private Subnet
│       │
│       └── EFS Mount Target
│
└── EC2 Instance
```

The mount target receives an IP address from the subnet.

Conceptually:

```text
Subnet
10.0.1.0/24
    │
    ▼
EFS Mount Target
10.0.1.x
```

---

# ⚠️ One Mount Target Per Availability Zone

For a Regional EFS filesystem, I can create:

```text
One Mount Target
per Availability Zone
```

I do **not** create one mount target for every subnet.

For example:

```text
AZ-A
│
├── Subnet A1
├── Subnet A2
└── Mount Target
      in one subnet
```

Instances in the AZ can access the filesystem through that mount target as long as networking and security allow it.

For high availability with a Regional filesystem, AWS recommends creating a mount target in each Availability Zone from which the filesystem will be accessed.

---

# 🌐 Connecting to EFS

EFS can be mounted using DNS.

Conceptually:

```text
EC2
 │
 ▼
EFS DNS Name
 │
 ▼
Mount Target
 │
 ▼
EFS File System
```

AWS provides DNS names associated with the filesystem and mount targets.

When using Regional EFS, workloads should normally access a mount target in their local Availability Zone.

This helps avoid unnecessary cross-AZ traffic and provides better architecture for highly available applications.

---

# 📡 Network File System Protocol

Amazon EFS uses:

```text
NFS
```

Amazon EFS supports NFS versions including:

```text
NFSv4.0

NFSv4.1
```

NFS is widely used by Linux and Unix-like systems for network file sharing.

A Linux server might mount EFS like:

```text
Linux EC2
    │
    ▼
NFS Client
    │
    ▼
EFS Mount Target
    │
    ▼
Amazon EFS
```

AWS also provides:

```text
amazon-efs-utils
```

which contains the Amazon EFS mount helper and additional functionality.

---

# 🐧 EFS and Linux

Amazon EFS is primarily designed for Linux and Unix-style workloads.

For example:

```text
Amazon Linux
Ubuntu
Red Hat Enterprise Linux
SUSE Linux
```

can access EFS using supported NFS clients.

---

# 🪟 What About Windows?

An important limitation is:

```text
Amazon EFS
     +
Microsoft Windows Client
     =
Not Supported
```

If I need managed shared file storage specifically for Windows workloads, I should evaluate services designed for that use case, such as:

```text
Amazon FSx for Windows File Server
```

This is an important architectural distinction.

---

# 🔐 POSIX Permissions

EFS uses standard Unix-style file permissions.

For example:

```text
Owner

Group

Others
```

with:

```text
Read
Write
Execute
```

A familiar Linux example:

```text
-rwxr-x---
```

Conceptually:

```text
EFS File
│
├── Owner Permissions
├── Group Permissions
└── Other Permissions
```

This makes EFS work naturally with Linux application environments.

---

# 🔒 Security Groups and EFS

Mount targets use security groups.

For example:

```text
EC2 Security Group
       │
       │ NFS TCP 2049
       ▼
EFS Security Group
       │
       ▼
Mount Target
```

A good pattern is:

```text
EFS Security Group

Inbound:
NFS TCP 2049

Source:
Application EC2 Security Group
```

instead of allowing NFS access from large network ranges unnecessarily.

---

# 🟡 EFS One Zone

Amazon EFS also provides:

```text
One Zone
```

file systems.

Unlike Regional EFS:

```text
Regional EFS
     │
     ├── AZ-A
     ├── AZ-B
     └── AZ-C
```

One Zone stores data within:

```text
One Availability Zone
```

---

# 🏗️ One Zone Architecture

```text
AWS Region
│
├── AZ-A
│    │
│    ├── EFS One Zone
│    │
│    └── Mount Target
│
└── AZ-B
```

A One Zone filesystem supports a **single mount target**, located in the same Availability Zone as the filesystem.

---

# 💰 Why Use One Zone?

The main reasons are:

```text
Lower Storage Cost

and

Workloads That Do Not Require
Multi-AZ Storage Resilience
```

Examples might include data that:

```text
Can Be Recreated

Has Another Copy

Does Not Require Multi-AZ Durability
```

---

# ⚠️ One Zone Availability Consideration

With One Zone:

```text
Availability Zone
       │
       ▼
EFS One Zone
```

The filesystem does not provide the same multi-AZ resilience as Regional EFS.

Therefore, I should choose it only when the workload can tolerate the availability and durability trade-off.

---

# 🌐 Accessing One Zone from Another AZ

Technically, a compute instance in another Availability Zone can access the One Zone mount target if network connectivity permits.

For example:

```text
AZ-B EC2
   │
   │ Cross-AZ
   ▼
AZ-A Mount Target
   │
   ▼
EFS One Zone
```

But this is generally not the preferred design.

It can introduce:

```text
Cross-AZ Data Transfer Charges

Higher Network Latency
```

For One Zone workloads, keeping compute in the same Availability Zone as the filesystem is normally preferable.

---

# 🆚 Regional vs One Zone

| Feature                      | Regional EFS                 | One Zone EFS                     |
| ---------------------------- | ---------------------------- | -------------------------------- |
| Data placement               | Multiple AZs                 | Single AZ                        |
| Mount targets                | Up to one per AZ             | One                              |
| Multi-AZ resilience          | ✅                            | ❌                                |
| Cost                         | Higher                       | Lower                            |
| Recommended for HA workloads | ✅                            | ❌                                |
| Best for                     | Production/shared HA storage | Cost-sensitive, recreatable data |

---

# 🏢 Accessing EFS from On-Premises

EFS is not limited to EC2 workloads inside the VPC.

Linux systems in an on-premises data center can access EFS using private connectivity such as:

```text
AWS Direct Connect

or

AWS Site-to-Site VPN
```

Architecture:

```text
On-Premises Linux
       │
       ▼
Direct Connect / VPN
       │
       ▼
Amazon VPC
       │
       ▼
EFS Mount Target
       │
       ▼
Amazon EFS
```

This can support hybrid architectures where on-premises Linux servers and AWS workloads need access to shared files.

---

# 🌐 EFS and VPCs

An EFS filesystem can have mount targets in:

```text
One VPC at a time
```

However, this does **not** mean only resources in that VPC can ever access it.

With appropriate networking, clients can access EFS through the mount targets from:

```text
Same VPC

Peered VPC

Connected Networks

On-Premises via VPN

On-Premises via Direct Connect
```

So I need to distinguish:

```text
Where Mount Targets Exist
```

from:

```text
Which Networks Can Reach Them
```

---

# 📈 Elastic Capacity

One major difference from EBS is capacity management.

With EBS:

```text
Create Volume
     │
     ▼
Specify Capacity
     │
     ▼
100 GiB
```

With EFS:

```text
Create File System
      │
      ▼
Write Data
      │
      ▼
Capacity Grows Automatically
      │
      ▼
Delete Data
      │
      ▼
Capacity Shrinks Automatically
```

I do not provision a fixed filesystem size before using EFS.

This is why the service is called:

```text
Elastic File System
```

---

# 💾 EFS Storage Classes

Amazon EFS provides storage classes designed for different access patterns.

A useful high-level view is:

```text
EFS Storage
│
├── Standard
├── Infrequent Access
└── Archive
```

One Zone file systems have corresponding One Zone storage behavior where applicable.

---

# 🟢 EFS Standard

EFS Standard is intended for frequently accessed data.

Examples:

```text
Active Application Files

Shared Web Content

Frequently Used Files

Application Data
```

It provides low-latency access for active workloads.

---

# 🟡 EFS Infrequent Access

EFS Infrequent Access is designed for files that are accessed less frequently.

Conceptually:

```text
Frequently Accessed
       │
       ▼
EFS Standard


Less Frequently Accessed
       │
       ▼
EFS Infrequent Access
```

It offers lower storage cost with an access charge when data is read or written.

---

# 🟠 EFS Archive

EFS Archive is designed for long-lived data that is rarely accessed.

For example:

```text
Old Application Data

Historical Files

Rarely Accessed Content
```

A simple progression is:

```text
Frequently Accessed
       │
       ▼
Standard
       │
       ▼
Less Frequently Accessed
       │
       ▼
Infrequent Access
       │
       ▼
Rarely Accessed
       │
       ▼
Archive
```

---

# 🔄 EFS Lifecycle Management

I do not necessarily need to manually decide which individual files belong in each storage class.

EFS Lifecycle Management can automatically transition files based on access patterns.

Conceptually:

```text
EFS Standard
     │
     │ Lifecycle Policy
     ▼
Infrequent Access
     │
     ▼
Archive
```

EFS can also transition files back toward Standard storage when configured appropriately and the files are accessed again.

This can help reduce storage costs without requiring applications to use a different filesystem.

---

# ⚡ EFS Throughput Modes

One important update from older training material is that EFS currently provides **three** throughput modes:

```text
EFS Throughput
│
├── Elastic
├── Provisioned
└── Bursting
```

---

# 🟢 Elastic Throughput

AWS currently recommends:

```text
Elastic Throughput
```

for most workloads.

With Elastic throughput:

```text
Application Demand
        │
        ▼
EFS Automatically
Scales Throughput
        │
        ▼
Workload Changes
        │
        ▼
Throughput Changes
```

This is particularly useful when workload throughput is:

```text
Unpredictable

Spiky

Difficult to Forecast
```

---

# 🔵 Provisioned Throughput

With Provisioned throughput, I specify the required throughput independently of filesystem size.

```text
EFS File System
       │
       ▼
Provisioned Throughput
       │
       ▼
Specified Performance Level
```

This can be useful when I know the workload's throughput requirement.

---

# 🟡 Bursting Throughput

With Bursting throughput, throughput scales based on the amount of data stored in the Standard storage class.

Conceptually:

```text
More Data Stored
      │
      ▼
Higher Baseline Throughput
```

Burst credits allow the filesystem to temporarily operate above its baseline throughput.

---

# 🆚 Throughput Modes

| Mode        | Best Fit                                      |
| ----------- | --------------------------------------------- |
| Elastic     | Unpredictable or spiky workloads              |
| Provisioned | Known throughput requirements                 |
| Bursting    | Throughput that should scale with stored data |

For most new workloads, AWS currently recommends starting with:

```text
Elastic Throughput
```

unless there is a specific reason to use another mode.

---

# ⚙️ EFS Performance Mode

Throughput mode should not be confused with:

```text
Performance Mode
```

Current EFS options include:

```text
General Purpose

Max I/O
```

AWS recommends:

```text
General Purpose
```

for current EFS workloads.

Max I/O is considered a previous-generation performance mode and has higher per-operation latency.

---

# 🔐 EFS Encryption

Amazon EFS also supports encryption.

There are two different areas to consider:

```text
Data at Rest

Data in Transit
```

---

# 🔒 Encryption at Rest

EFS integrates with:

```text
AWS KMS
```

to protect data stored in the filesystem.

Conceptually:

```text
Application
    │
    ▼
Amazon EFS
    │
    ▼
Encrypted Data at Rest
    │
    ▼
AWS KMS
```

Encryption at rest should be considered when creating the filesystem because the encryption setting cannot simply be changed on an existing filesystem later.

---

# 🔐 Encryption in Transit

Traffic between clients and EFS can also be protected using TLS.

The Amazon EFS mount helper simplifies encrypted mounting.

Conceptually:

```text
EC2
 │
 │ TLS
 ▼
EFS Mount Target
 │
 ▼
Amazon EFS
```

This helps protect data while it moves across the network.

---

# 🏗️ Example Highly Available Web Architecture

A common EFS use case is shared application content.

```text
                    Internet
                       │
                       ▼
             Application Load Balancer
                       │
              ┌────────┴────────┐
              ▼                 ▼
           EC2-A               EC2-B
           AZ-A                AZ-B
              │                 │
              └────────┬────────┘
                       ▼
                  Amazon EFS
                       │
                  Shared Files
```

Both application servers can access the same files.

This is very different from storing independent copies on separate EBS volumes.

---

# 📦 Other EFS Use Cases

EFS can be useful for:

```text
Shared Web Content

Content Management Systems

Home Directories

Development Environments

Shared Application Files

Container Persistent Storage

Analytics Workloads

Machine Learning Workloads
```

EFS can also integrate with AWS compute services beyond EC2, including appropriate container and serverless architectures.

---

# 🧠 EBS vs EFS Decision

A useful decision process is:

```text
What Does the Application Need?
            │
      ┌─────┴─────┐
      │           │
      ▼           ▼
Block Storage   Shared Files
      │           │
      ▼           ▼
     EBS         EFS
```

For example:

```text
EC2 Root Disk
      │
      ▼
     EBS


Database Block Storage
      │
      ▼
     EBS


Multiple Linux Servers
Need Same Files
      │
      ▼
     EFS
```

---

# ⚠️ Common Mistakes

### Mistake 1: Treating EFS Like EBS

EBS is block storage.

EFS is shared file storage accessed over NFS.

---

### Mistake 2: Thinking EFS Has a Fixed Capacity

EFS automatically grows and shrinks as files are added and removed.

---

### Mistake 3: Creating a Mount Target in Every Subnet

For a Regional filesystem, there can be only one mount target per Availability Zone.

---

### Mistake 4: Using Windows Clients with EFS

Microsoft Windows clients are not supported for Amazon EFS.

For Windows file-sharing workloads, evaluate the appropriate Amazon FSx offering.

---

### Mistake 5: Using One Zone Without Understanding the Risk

One Zone does not provide the same multi-AZ resilience as Regional EFS.

Use it only when the workload can tolerate that trade-off.

---

### Mistake 6: Opening NFS to Everyone

Do not unnecessarily configure:

```text
TCP 2049
Source: 0.0.0.0/0
```

Prefer security-group-to-security-group access where appropriate.

---

### Mistake 7: Thinking There Are Only Two Throughput Modes

Current EFS provides:

```text
Elastic

Provisioned

Bursting
```

Elastic is AWS's recommended choice for most new workloads.

---

### Mistake 8: Confusing Performance Mode and Throughput Mode

These are separate settings.

```text
Performance Mode
       │
       └── General Purpose


Throughput Mode
       │
       ├── Elastic
       ├── Provisioned
       └── Bursting
```

---

# ✅ Best Practices

* Use Regional EFS when applications require multi-AZ resilience.
* Create Regional mount targets in each Availability Zone from which the filesystem is accessed.
* Keep compute close to its local mount target where possible.
* Use One Zone only when the availability and durability trade-off is acceptable.
* Use security groups to restrict NFS access.
* Use encryption at rest for sensitive data.
* Use TLS when encryption in transit is required.
* Start with Elastic throughput for unpredictable workloads unless requirements indicate another mode.
* Use General Purpose performance mode for current workloads.
* Configure Lifecycle Management when infrequently accessed files can move to lower-cost storage.
* Use consistent POSIX ownership and permissions.
* Monitor EFS performance and throughput using Amazon CloudWatch.
* Test failure scenarios when EFS is part of a highly available application.

---

# ❓ Interview Questions

### Q1. What is Amazon EFS?

Amazon EFS is a managed elastic file-storage service that provides shared filesystem access using NFS.

---

### Q2. What is the main difference between EBS and EFS?

```text
EBS
 │
 └── Block Storage


EFS
 │
 └── Shared File Storage
```

---

### Q3. Is EFS Regional?

EFS provides both:

```text
Regional

and

One Zone
```

filesystem types.

Regional EFS stores data across multiple Availability Zones.

One Zone stores data within one Availability Zone.

---

### Q4. What is an EFS mount target?

A mount target provides a network endpoint through which clients access an EFS filesystem.

---

### Q5. How many Regional EFS mount targets can I create in an Availability Zone?

One mount target per Availability Zone.

---

### Q6. What protocol does EFS use?

Network File System:

```text
NFS
```

including NFSv4.x support.

---

### Q7. Can Windows EC2 instances use Amazon EFS as a supported client?

No.

Microsoft Windows-based clients are not supported by Amazon EFS.

---

### Q8. What port does NFS use with EFS?

```text
TCP 2049
```

---

### Q9. Does EFS require me to provision storage capacity?

No.

EFS storage capacity automatically grows and shrinks as data is added and removed.

---

### Q10. What is the difference between Regional and One Zone EFS?

Regional EFS stores data across multiple Availability Zones.

One Zone stores data in a single Availability Zone and costs less but provides lower resilience to AZ-level failure.

---

### Q11. Can an EC2 instance in another AZ access a One Zone filesystem?

Yes, if network connectivity permits, but keeping compute in the same AZ is generally preferable because cross-AZ access can add latency and data-transfer charges.

---

### Q12. Can on-premises Linux servers access EFS?

Yes.

They can access EFS through private connectivity such as:

```text
AWS Direct Connect

or

AWS Site-to-Site VPN
```

provided the required routing, security, and NFS configuration exists.

---

### Q13. What are the EFS throughput modes?

```text
Elastic

Provisioned

Bursting
```

---

### Q14. Which throughput mode does AWS recommend for most workloads?

Elastic throughput.

It automatically scales throughput according to workload requirements.

---

### Q15. What are EFS storage classes used for?

They provide different cost models based on file-access frequency.

Examples include:

```text
Standard

Infrequent Access

Archive
```

---

### Q16. What does EFS Lifecycle Management do?

It can automatically transition files between storage classes based on configured lifecycle policies.

---

### Q17. Can EFS mount targets exist in multiple VPCs simultaneously?

No.

A filesystem can have mount targets in only one VPC at a time.

Cl
