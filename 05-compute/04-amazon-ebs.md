# 💾 Amazon Elastic Block Store (Amazon EBS)

> Amazon Elastic Block Store (Amazon EBS) provides persistent block-level storage volumes that can be attached to Amazon EC2 instances.

---

# 📖 Overview

An EC2 instance is a virtual server.

Just like a physical computer requires storage for its operating system, applications, and data, an EC2 instance often requires persistent block storage.

Amazon Elastic Block Store provides this capability.

```text
EC2 Instance
     │
     ▼
EBS Volume
     │
     ├── Operating System
     ├── Applications
     └── Data
```

An EBS volume behaves similarly to a block storage device attached to a server.

After attaching an EBS volume to an EC2 instance, the operating system sees it as a block device that can be partitioned, formatted with a file system, mounted, and used by applications.

---

# 🎯 Why Amazon EBS?

EC2 applications commonly require storage that:

* Persists independently from the running life of an instance
* Supports frequent reads and writes
* Can store operating systems
* Can store application data
* Can support database workloads
* Can be backed up
* Can be resized or have performance characteristics modified

Amazon EBS provides persistent block storage designed for these requirements.

---

# 🧱 What is Block Storage?

Block storage divides data into fixed-size blocks that an operating system can use as a storage device.

Conceptually:

```text
Application
     │
     ▼
File System
     │
     ▼
Operating System
     │
     ▼
Block Device
     │
     ▼
EBS Volume
```

The operating system can create a file system on the volume.

Examples include:

```text
Linux
  │
  ├── ext4
  └── XFS

Windows
  │
  └── NTFS
```

From the operating system's perspective, the attached EBS volume behaves much like a disk.

---

# 🏗️ EC2 and EBS Architecture

Consider an EC2 instance running inside a VPC.

```text
AWS Region
│
├── Availability Zone A
│
│   ├── Subnet A
│   │     │
│   │     └── EC2-A
│   │
│   └── EBS Volume-A
│
└── Availability Zone B
    │
    ├── Subnet B
    │     │
    │     └── EC2-B
    │
    └── EBS Volume-B
```

The EC2 instance accesses its EBS volume through AWS storage infrastructure.

The volume appears to the EC2 operating system as a block device.

---

# 🌎 EBS Volumes are Availability Zone Specific

One of the most important characteristics of Amazon EBS is:

```text
EBS Volume
     │
     ▼
Belongs to one
Availability Zone
```

An EBS volume can be attached only to EC2 instances located in the **same Availability Zone**.

Example:

```text
Availability Zone A

EC2-A
  │
  ▼
EBS-A

✅ Supported
```

But:

```text
AZ-A                         AZ-B

EBS-A  ───────────────────► EC2-B

                ❌
```

An EBS volume in AZ-A cannot be directly attached to an EC2 instance in AZ-B.

---

# 🧠 Why Does the Availability Zone Matter?

Suppose we have:

```text
Region
│
├── AZ-A
│   ├── EC2-A
│   └── EBS-A
│
└── AZ-B
    ├── EC2-B
    └── EBS-B
```

The correct relationships are:

```text
EC2-A → EBS-A

EC2-B → EBS-B
```

Not:

```text
EC2-A → EBS-B
```

Therefore, when creating an EBS volume for an existing EC2 instance, one of the first things to check is:

```text
Which Availability Zone
is the EC2 instance in?
```

Then create the EBS volume in that same AZ.

---

# 🛡️ EBS Volume Durability

Although an EBS volume belongs to a single Availability Zone, it is not stored as only one physical copy on one storage device.

AWS automatically replicates EBS volume data **within its Availability Zone** to protect against failures of individual hardware components.

Conceptually:

```text
Availability Zone A
│
└── EBS Service
      │
      ├── Storage Infrastructure
      ├── Replication
      └── Hardware Failure Protection
```

This provides durability against individual component failures.

However:

```text
Replication within AZ
        ≠
Cross-AZ backup
```

For protection beyond the volume's AZ, EBS snapshots become important.

---

# 💿 Root EBS Volume

An EC2 instance can use an EBS volume as its root device.

```text
EC2 Instance
     │
     ▼
Root EBS Volume
     │
     ├── Operating System
     ├── System Files
     └── Applications
```

For an EBS-backed EC2 instance, this volume contains the operating system used to boot the instance.

---

# 💾 Additional EBS Volumes

An EC2 instance can also have additional EBS data volumes.

Example:

```text
EC2 Instance
     │
     ├── Root Volume
     │     └── Operating System
     │
     ├── Data Volume 1
     │     └── Application Data
     │
     └── Data Volume 2
           └── Database Data
```

The maximum number of volumes that can be attached depends on the EC2 instance type.

---

# 🔄 EBS Volume Lifecycle

An EBS volume has its own lifecycle.

Conceptually:

```text
Create Volume
     │
     ▼
Available
     │
     ▼
Attach to EC2
     │
     ▼
In Use
     │
     ▼
Detach
     │
     ▼
Available
     │
     ├── Attach to another EC2
     │
     └── Delete
```

Because the volume exists independently from the EC2 instance, it can be detached and attached to another compatible EC2 instance in the same Availability Zone.

---

# 🔄 Moving an EBS Volume Between Instances

Consider:

```text
Availability Zone A

EC2-A
  │
  ▼
EBS Volume
```

The volume can be detached from EC2-A.

```text
EC2-A

EBS Volume
```

Then attached to another EC2 instance in the same AZ:

```text
Availability Zone A

EC2-B
  │
  ▼
EBS Volume
```

This can be useful for:

* Data recovery
* Troubleshooting
* Server replacement
* Data migration between instances in the same AZ

---

# 🧠 EC2 and EBS Lifecycle Independence

An important concept is:

```text
EC2 Instance Lifecycle
          ≠
EBS Volume Lifecycle
```

For example:

```text
EC2-A
  │
  ▼
EBS Data Volume
```

If EC2-A becomes unusable, the data volume can potentially be detached and attached to another instance:

```text
EC2-A
 Failed
   ✗

EBS Volume
    │
    ▼
EC2-B
Healthy
```

This separation between compute and storage is an important cloud architecture principle.

The exact behavior when terminating an EC2 instance also depends on the volume's **DeleteOnTermination** setting.

---

# ⚙️ EBS Volume Types

Amazon EBS provides different volume types optimized for different workload requirements.

The main families include:

```text
EBS Volumes
│
├── SSD
│   │
│   ├── General Purpose SSD
│   │     ├── gp3
│   │     └── gp2
│   │
│   └── Provisioned IOPS SSD
│         ├── io2
│         └── io1
│
└── HDD
    │
    ├── Throughput Optimized HDD
    │     └── st1
    │
    └── Cold HDD
          └── sc1
```

Different volume types provide different combinations of:

```text
IOPS
  +
Throughput
  +
Latency Characteristics
  +
Capacity
  +
Cost
```

Choosing an EBS volume should therefore be based on the workload's storage requirements.

The individual EBS volume types should be covered separately in more detail.

---

# ⚡ IOPS vs Throughput

Two important storage-performance concepts are:

```text
IOPS

and

Throughput
```

### IOPS

IOPS means:

```text
Input / Output Operations Per Second
```

It measures how many storage operations can be performed each second.

### Throughput

Throughput measures the amount of data transferred over a period of time.

Conceptually:

```text
IOPS
 │
 └── How many operations?


Throughput
 │
 └── How much data?
```

Different applications may need different combinations of these characteristics.

---

# 🔗 EBS Multi-Attach

Normally, an EBS volume is attached to one EC2 instance.

```text
EC2
 │
 ▼
EBS
```

Amazon EBS also provides a feature called:

```text
EBS Multi-Attach
```

Multi-Attach allows a supported EBS volume to be attached to multiple EC2 instances simultaneously.

```text
EC2-A ──┐
        │
EC2-B ──┼──► EBS Volume
        │
EC2-C ──┘
```

However, Multi-Attach is not available for every EBS volume type.

It is supported for:

```text
io1
io2
```

Provisioned IOPS SSD volumes, subject to AWS's instance, operating system, Region, and volume-type requirements.

---

# ⚠️ Multi-Attach Data Consistency

Multi-Attach gives multiple EC2 instances read/write access to the same block device.

That creates an important challenge:

```text
EC2-A
   │
   │ Write
   ▼
Shared EBS
   ▲
   │ Write
   │
EC2-B
```

The applications and file-system architecture must coordinate concurrent access correctly.

EBS does not automatically turn a Multi-Attach block device into a normal shared network file system.

The workload must use a file system or application architecture designed to coordinate access from multiple servers.

---

# 🧠 Multi-Attach vs Shared File Storage

Do not assume:

```text
EBS Multi-Attach
        =
Shared File System
```

They solve different problems.

For workloads requiring shared file access across multiple servers, a managed shared file system such as Amazon EFS may be more appropriate depending on the application requirements.

---

# 📸 Amazon EBS Snapshots

Amazon EBS provides snapshots for backing up EBS volumes.

A snapshot is a:

```text
Point-in-Time Backup
of an
EBS Volume
```

Architecture:

```text
EBS Volume
     │
     │ Create Snapshot
     ▼
EBS Snapshot
```

Snapshots can later be used to create new EBS volumes.

---

# 🧠 EBS Snapshots are Incremental

EBS snapshots are incremental.

After the first snapshot, only changed blocks need to be stored in subsequent snapshots.

Conceptually:

```text
EBS Volume
     │
     ├── Snapshot 1
     │      └── Initial blocks
     │
     ├── Snapshot 2
     │      └── Changed blocks
     │
     └── Snapshot 3
            └── Changed blocks
```

Even though snapshots are incremental internally, each snapshot contains the information required to restore the volume to the state represented by that snapshot.

---

# 🪣 Where are EBS Snapshots Stored?

EBS snapshots are stored using Amazon S3 infrastructure in AWS-managed S3 buckets.

However:

```text
You do not manage
or directly access
those S3 buckets
```

You manage the snapshots through services and interfaces such as:

```text
Amazon EC2 Console

AWS CLI

AWS APIs

AWS Backup

Amazon Data Lifecycle Manager
```

Do not expect the EBS snapshot to appear as an object inside one of your own normal S3 buckets.

---

# 🌎 Snapshot Availability

A normal EBS volume belongs to:

```text
One Availability Zone
```

A standard regional EBS snapshot belongs to:

```text
One AWS Region
```

Snapshot data is automatically replicated across Availability Zones within that Region.

This distinction is extremely important:

```text
EBS Volume
     │
     └── Availability Zone Scoped


EBS Snapshot
     │
     └── Region Scoped
```

---

# 🔄 Create a Volume in Another Availability Zone

Suppose:

```text
AZ-A

EC2-A
  │
  ▼
EBS-A
```

You cannot directly attach `EBS-A` to an EC2 instance in AZ-B.

Instead:

```text
EBS-A
  │
  ▼
Snapshot
  │
  ▼
Create New Volume
in AZ-B
  │
  ▼
EBS-B
  │
  ▼
EC2-B
```

This allows EBS data to be restored into another Availability Zone.

---

# 🛡️ Availability Zone Recovery

Consider an application running in:

```text
Availability Zone A

EC2-A
  │
  ▼
EBS-A
```

If you need to recover the data into another AZ, an EBS snapshot can be used to create a new volume there:

```text
Snapshot
   │
   ▼
Create EBS-B
in AZ-B
   │
   ▼
Attach to EC2-B
```

This makes snapshots an important component of backup and recovery architecture.

However, snapshots must actually be created.

AWS does **not** automatically create regular EBS snapshots for your volumes simply because you use EBS.

Backup automation can be implemented using services such as:

```text
AWS Backup

or

Amazon Data Lifecycle Manager
```

---

# 🌍 Copying Snapshots Across Regions

EBS snapshots can also be copied between AWS Regions.

Example:

```text
Region A
   │
   │
EBS Volume
   │
   ▼
Snapshot
   │
   │ Copy Snapshot
   ▼
Region B
   │
   ▼
Snapshot Copy
   │
   ▼
New EBS Volume
```

Once the snapshot exists in the destination Region, it can be used to create an EBS volume in an Availability Zone within that Region.

---

# 🛡️ Disaster Recovery

Cross-Region snapshot copies can support disaster recovery strategies.

Example:

```text
Primary Region
ca-central-1
      │
      ▼
EBS Volume
      │
      ▼
Snapshot
      │
      │ Copy
      ▼
Recovery Region
us-east-1
      │
      ▼
Snapshot
      │
      ▼
EBS Volume
      │
      ▼
Recovery EC2
```

This does not by itself create a complete disaster recovery solution.

The architecture must also consider:

* Compute
* Networking
* IAM
* Application configuration
* Databases
* DNS
* Recovery procedures
* Recovery Point Objective (RPO)
* Recovery Time Objective (RTO)

But EBS snapshots can provide the block-storage recovery component.

---

# 📊 EBS Volume vs EBS Snapshot

| Characteristic             | EBS Volume                      | EBS Snapshot                    |
| -------------------------- | ------------------------------- | ------------------------------- |
| Purpose                    | Active block storage            | Point-in-time backup            |
| Scope                      | Availability Zone               | Region                          |
| Attach directly to EC2     | Yes                             | No                              |
| Create volume from it      | N/A                             | Yes                             |
| Cross-AZ recovery          | Cannot directly move attachment | Create new volume from snapshot |
| Cross-Region               | Volume itself remains AZ-scoped | Snapshot can be copied          |
| Used by OS as block device | Yes                             | No                              |
| Incremental                | No                              | Snapshots are incremental       |

---

# 🔄 EBS Architecture Summary

```text
                    AWS Region
                        │
          ┌─────────────┴─────────────┐
          │                           │
          ▼                           ▼
        AZ-A                        AZ-B
          │                           │
          ▼                           ▼
       EC2-A                        EC2-B
          │                           ▲
          ▼                           │
       EBS-A                          │
          │                           │
          │ Snapshot                  │
          ▼                           │
     EBS Snapshot                     │
          │                           │
          └──── Create Volume ────────┘
```

The key relationship is:

```text
Volume
  │
  └── AZ Scoped

Snapshot
  │
  └── Region Scoped
```

---

# ⚠️ Common Mistakes

### Mistake 1: Creating the Volume in the Wrong AZ

```text
EC2
AZ-A

EBS
AZ-B

❌ Cannot attach
```

Always verify the Availability Zone before creating the volume.

---

### Mistake 2: Assuming EBS is Instance-Local Storage

EBS is persistent block storage provided separately from the EC2 host.

Do not confuse:

```text
Amazon EBS

with

EC2 Instance Store
```

Instance Store should be covered separately.

---

### Mistake 3: Assuming Every EBS Volume Supports Multi-Attach

Multi-Attach is supported only for specific Provisioned IOPS SSD volume types and configurations.

---

### Mistake 4: Treating Multi-Attach Like EFS

Multi-Attach provides shared access to a block device.

It does not automatically provide the file-sharing semantics of a managed network file system.

---

### Mistake 5: Assuming Snapshots Appear in Your S3 Bucket

EBS snapshots use AWS-managed S3 infrastructure.

You do not directly manage the underlying S3 buckets.

---

### Mistake 6: Assuming EBS Snapshots Happen Automatically

Regular snapshots must be created manually or automated.

For automated backup management, consider:

```text
Amazon Data Lifecycle Manager

or

AWS Backup
```

---

### Mistake 7: Confusing Volume Scope and Snapshot Scope

Remember:

```text
EBS Volume
    =
Availability Zone


EBS Snapshot
    =
Region
```

---

# ✅ Best Practices

* Keep EBS volumes in the same Availability Zone as the EC2 instances that use them.
* Select the EBS volume type based on workload IOPS, throughput, latency, capacity, and cost requirements.
* Use separate data volumes when appropriate rather than storing everything on the root volume.
* Enable EBS encryption according to security requirements.
* Create regular snapshots for important data.
* Automate snapshot lifecycle management using AWS Backup or Amazon Data Lifecycle Manager where appropriate.
* Test restoration procedures rather than assuming backups are recoverable.
* Copy snapshots to another Region when the disaster recovery requirements justify it.
* Monitor EBS performance using Amazon CloudWatch.
* Understand DeleteOnTermination settings before terminating EC2 instances.
* Use Multi-Attach only for applications and file systems designed for coordinated concurrent access.
* Delete unused EBS volumes and snapshots according to retention requirements to control costs.

---

# ❓ Interview Questions

### Q1. What is Amazon EBS?

**Answer**

Amazon Elastic Block Store provides persistent block-level storage volumes that can be attached to Amazon EC2 instances.

---

### Q2. Is an EBS volume tied to a Region or Availability Zone?

**Answer**

An EBS volume is created in a specific Availability Zone.

---

### Q3. Can an EBS volume in AZ-A be attached directly to an EC2 instance in AZ-B?

**Answer**

No.

The EC2 instance and EBS volume must be in the same Availability Zone.

---

### Q4. Can multiple EBS volumes be attached to one EC2 instance?

**Answer**

Yes.

The number of EBS volumes that can be attached depends on the EC2 instance type.

---

### Q5. Can an EBS volume be detached from one EC2 instance and attached to another?

**Answer**

Yes, provided the destination instance is compatible and located in the same Availability Zone as the volume.

---

### Q6. Does EBS data survive independently of an EC2 instance?

**Answer**

EBS volumes have a lifecycle independent from the running EC2 instance.

Whether a particular EBS volume is deleted when an EC2 instance is terminated depends on settings such as `DeleteOnTermination`.

---

### Q7. What is EBS Multi-Attach?

**Answer**

EBS Multi-Attach allows a supported Provisioned IOPS SSD volume to be attached to multiple supported EC2 instances in the same Availability Zone.

---

### Q8. How many instances can use a Multi-Attach enabled volume?

**Answer**

AWS currently supports attaching a Multi-Attach enabled volume to up to **16 Nitro-based instances in the same Availability Zone**, subject to the supported volume type and operating-system requirements.

---

### Q9. What is an EBS snapshot?

**Answer**

An EBS snapshot is a point-in-time backup of an EBS volume.

---

### Q10. Are EBS snapshots incremental?

**Answer**

Yes.

After the initial snapshot, subsequent snapshots store changed blocks incrementally.

---

### Q11. Where are EBS snapshots stored?

**Answer**

EBS snapshots are stored using Amazon S3 infrastructure in AWS-managed buckets that customers do not directly access.

---

### Q12. Can a snapshot be used to create an EBS volume in another Availability Zone?

**Answer**

Yes.

A regional snapshot can be used to create a new EBS volume in another Availability Zone within the same Region.

---

### Q13. How can EBS data be moved to another Region?

**Answer**

One common approach is:

```text
EBS Volume
    │
    ▼
Create Snapshot
    │
    ▼
Copy Snapshot
to Another Region
    │
    ▼
Create New EBS Volume
```

---

### Q14. What is the difference between an EBS volume and an EBS snapshot?

**Answer**

An EBS volume is active block storage attached to EC2 and is Availability Zone scoped.

An EBS snapshot is a point-in-time backup and, for standard regional snapshots, is Region scoped.

---

### Q15. Does AWS automatically create regular snapshots of every EBS volume?

**Answer**

No.

Snapshot creation should be configured according to the workload's backup requirements, either manually or through automation such as AWS Backup or Amazon Data Lifecycle Manager.

---

### Q16. What should you check before creating an EBS volume for an existing EC2 instance?

**Answer**

At minimum, identify:

```text
EC2 Availability Zone

Required Capacity

Required IOPS

Required Throughput

Volume Type

Encryption Requirements

Backup Requirements
```

The EBS volume must be created in the same Availability Zone as the EC2 instance that will use it.

---

# 💡 Key Takeaways

* Amazon EBS provides persistent block-level storage for EC2.
* EBS can be used for operating systems, applications, databases, and application data.
* EBS volumes are Availability Zone scoped.
* An EC2 instance and its attached EBS volume must be in the same Availability Zone.
* EBS data is automatically replicated within its Availability Zone to protect against individual hardware failures.
* Multiple EBS volumes can be attached to one EC2 instance, subject to instance limits.
* EBS volumes have a lifecycle independent of running EC2 instances.
* A volume can be detached and attached to another compatible instance in the same AZ.
* Different EBS volume types provide different IOPS, throughput, capacity, and cost characteristics.
* Multi-Attach allows supported Provisioned IOPS SSD volumes to be attached to multiple supported instances in the same AZ.
* Applications using Multi-Attach must correctly coordinate concurrent access.
* EBS snapshots provide point-in-time backups.
* EBS snapshots are incremental.
* Standard regional EBS snapshot data is replicated across Availability Zones in the Region.
* A snapshot can be used to create a new volume in another Availability Zone.
* Snapshots can be copied to other AWS Regions.
* Cross-Region snapshot copies can form part of a disaster recovery strategy.
* Backups should be automated and recovery procedures should be tested.

---

# 📚 Related Topics

* Amazon EC2
* EC2 Instance Types
* EC2 Instance Store
* EBS Volume Types
* EBS Encryption
* EBS Snapshots
* Amazon Data Lifecycle Manager
* AWS Backup
* Amazon EFS
* Amazon S3
* Amazon CloudWatch
* Disaster Recovery on AWS

---

# 📖 References

* AWS Documentation: Amazon EBS Volumes
* AWS Documentation: Features and Benefits of Amazon EBS
* AWS Documentation: Attach an Amazon EBS Volume to an EC2 Instance
* AWS Documentation: Amazon EBS Volume Lifecycle
* AWS Documentation: Amazon EBS Snapshots
* AWS Documentation: Copy an Amazon EBS Snapshot
* AWS Documentation: Amazon EBS Multi-Attach
