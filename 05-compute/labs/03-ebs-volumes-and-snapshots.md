# 💾 Lab 03: Amazon EBS Volumes and Snapshots

> Create and attach EBS data volumes to Linux and Windows EC2 instances, configure the operating system to use them, create snapshots, restore data from snapshots, and copy snapshots across AWS Regions.

---

# 📖 Lab Overview

In the previous lessons, I learned the concepts behind:

```text
Amazon EBS
    │
    ├── Persistent Block Storage
    │
    ├── EBS Volume Types
    │
    ├── gp2 / gp3
    │
    ├── Provisioned IOPS
    │
    └── EBS Snapshots
```

This lab puts those concepts into practice.

The main goal is to understand the complete EBS lifecycle:

```text
Create
   │
   ▼
Attach
   │
   ▼
Format
   │
   ▼
Mount
   │
   ▼
Store Data
   │
   ▼
Snapshot
   │
   ▼
Restore
   │
   ▼
Validate Data
```

I will perform the exercise first using Linux and then Windows.

Finally, I will create a snapshot and use it to recover the data.

---

# 🎯 Learning Objectives

By the end of this lab, I should be able to explain and demonstrate:

* How to create an EBS data volume
* Why an EBS volume must be in the same Availability Zone as the EC2 instance
* How to attach an EBS volume to an EC2 instance
* How Linux detects EBS block devices
* How to create a Linux filesystem
* How to mount an EBS volume
* How `/etc/fstab` makes the mount persistent
* How Windows detects and initializes a new EBS disk
* How to create an NTFS filesystem
* How to create an EBS snapshot
* How to restore an EBS volume from a snapshot
* How to copy an EBS snapshot to another AWS Region
* How EBS enables data recovery independent of the original EC2 instance
* How to troubleshoot common EBS attachment and mounting problems

---

# 🏗️ Lab Architecture

The first part of the lab uses a Linux EC2 instance.

```text
AWS Region
│
└── VPC
    │
    └── Availability Zone A
        │
        ├── Public Subnet
        │      │
        │      └── Linux EC2
        │             │
        │             ├── Root EBS
        │             │
        │             └── Data EBS
        │                  gp3
        │
        └── EBS Volume
             Same AZ
```

The Windows exercise follows the same storage concept:

```text
Windows EC2
    │
    ├── C:
    │   Root EBS Volume
    │
    └── D:
        Data EBS Volume
```

The final exercise introduces snapshots:

```text
EBS Data Volume
      │
      ▼
EBS Snapshot
      │
      ├───────────────┐
      │               │
      ▼               ▼
New Volume       Copy Snapshot
Same Region      Another Region
      │               │
      ▼               ▼
EC2 Instance      New EBS Volume
                      │
                      ▼
                  EC2 Instance
```

---

# 📋 Prerequisites

Before starting, I need:

```text
AWS Account

Existing VPC

Public Subnet

Internet Gateway / Routing

Security Group

EC2 permissions

EBS permissions
```

For Linux access, I can use:

```text
EC2 Instance Connect

or

AWS Systems Manager Session Manager
```

if the environment is configured appropriately.

For Windows, I need an appropriate secure administrative access method.

If RDP is used, TCP port `3389` should be restricted to my trusted source IP rather than:

```text
0.0.0.0/0
```

---

# 🧪 Part 1: Linux EBS Data Volume

## Phase 1: Launch the Linux EC2 Instance

Launch an Amazon Linux EC2 instance.

Example:

```text
Name:
Linux-Server-01

AMI:
Amazon Linux 2023

Network:
Existing VPC

Subnet:
Public Subnet

Public IPv4:
Enabled

Security Group:
Existing administrative access SG
```

For this lab, a small instance type is sufficient.

---

# 🔍 Record the Availability Zone

After the instance starts, record its Availability Zone.

Example:

```text
Linux-Server-01

Availability Zone:
ca-central-1a
```

The exact AZ will depend on my environment.

This value is important because:

```text
EC2 Instance AZ
       =
EBS Volume AZ
```

for an EBS volume to be attached to that instance.

---

# 💾 Phase 2: Create a GP3 Data Volume

Navigate to:

```text
EC2
  │
  ▼
Elastic Block Store
  │
  ▼
Volumes
```

Create a new volume.

Example configuration:

```text
Volume Type:
gp3

Size:
10 GiB

Availability Zone:
Same AZ as Linux-Server-01

Snapshot:
None
```

Add a useful tag:

```text
Name = Linux-Data-Volume
```

After creation, the volume should enter:

```text
available
```

state.

This means:

> The EBS volume exists but is not currently attached to an EC2 instance.

---

# 🧠 Important: EBS is Availability Zone Specific

Suppose the EC2 instance is:

```text
ca-central-1a
```

Then the volume must also be:

```text
ca-central-1a
```

This will not work:

```text
EC2
ca-central-1a

      X

EBS
ca-central-1b
```

The EBS volume must first exist in the same AZ as the target EC2 instance.

---

# 🔗 Phase 3: Attach the EBS Volume

Select:

```text
Linux-Data-Volume
```

Then:

```text
Actions
   │
   ▼
Attach Volume
```

Select:

```text
Linux-Server-01
```

Use one of the recommended device names shown by AWS.

After attaching, the volume state changes from:

```text
available

    ↓

in-use
```

---

# 🖥️ Phase 4: Connect to Linux

Connect to:

```text
Linux-Server-01
```

using the access method configured for the environment.

Once connected:

```bash
whoami
hostname
```

Now inspect the available block devices:

```bash
lsblk
```

Example conceptual output:

```text
NAME        SIZE TYPE MOUNTPOINTS
nvme0n1       8G disk
└─nvme0n1p1   8G part /
nvme1n1      10G disk
```

The exact Linux device name can vary.

This is important.

> Do not assume that the device name requested in the EC2 console is the same device name Linux will display.

Nitro-based EC2 instances commonly expose EBS volumes as NVMe devices.

---

# 🔍 Phase 5: Identify the New Volume

Compare:

```bash
lsblk
```

with the EBS volumes shown in the EC2 console.

The new disk should approximately match the size of the volume created earlier:

```text
10 GiB
```

Before formatting anything, confirm that I have identified the correct disk.

This is extremely important because formatting the wrong device could destroy existing data.

---

# 🗂️ Phase 6: Check for an Existing Filesystem

Before formatting the disk, check whether it already contains a filesystem.

Example:

```bash
sudo file -s /dev/<device>
```

For a newly created empty volume, it should not contain an existing filesystem.

If the volume came from a snapshot and already contains data:

```text
DO NOT FORMAT IT
```

Formatting would destroy the existing filesystem and data.

---

# 🛠️ Phase 7: Create the Filesystem

For this new empty lab volume, create an EXT4 filesystem:

```bash
sudo mkfs.ext4 /dev/<device>
```

Replace:

```text
<device>
```

with the actual device identified using `lsblk`.

For example:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

The filesystem is now created, but it is not yet mounted.

---

# 📁 Phase 8: Create a Mount Point

Create a directory:

```bash
sudo mkdir -p /mnt/data
```

This directory becomes the location through which the operating system accesses the EBS filesystem.

Conceptually:

```text
EBS Volume
    │
    ▼
Block Device
    │
    ▼
EXT4 Filesystem
    │
    ▼
/mnt/data
```

---

# 🔗 Phase 9: Mount the Volume

Mount the filesystem:

```bash
sudo mount /dev/<device> /mnt/data
```

Example:

```bash
sudo mount /dev/nvme1n1 /mnt/data
```

Verify:

```bash
df -h
```

Also check:

```bash
lsblk
```

I should now see:

```text
/mnt/data
```

associated with the new device.

---

# ✍️ Phase 10: Write Test Data

Create a test file:

```bash
echo "EBS Lab - Persistent Data" | sudo tee /mnt/data/aws-data.txt
```

Verify:

```bash
cat /mnt/data/aws-data.txt
```

Expected:

```text
EBS Lab - Persistent Data
```

The file is now stored on the EBS data volume.

---

# 🧠 Mounting is Currently Temporary

At this point:

```text
EBS Attached
     │
     ▼
Filesystem Created
     │
     ▼
Mounted at /mnt/data
```

However, Linux does not yet know that it should automatically mount this filesystem after a reboot.

We need:

```text
/etc/fstab
```

for persistent mounting.

---

# 🔎 Phase 11: Find the Filesystem UUID

Run:

```bash
sudo blkid
```

or:

```bash
sudo blkid /dev/<device>
```

Example conceptual output:

```text
/dev/nvme1n1:
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
TYPE="ext4"
```

Copy the UUID.

Using the UUID is preferable to relying on a device name because device names can change.

---

# ⚙️ Phase 12: Configure `/etc/fstab`

First back up the existing configuration:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Edit:

```bash
sudo nano /etc/fstab
```

Add an entry similar to:

```text
UUID=<volume-uuid> /mnt/data ext4 defaults,nofail 0 2
```

For example:

```text
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/data ext4 defaults,nofail 0 2
```

Save the file.

---

# 🧪 Phase 13: Validate `/etc/fstab`

Before rebooting, validate the configuration.

Run:

```bash
sudo mount -a
```

If there are errors:

```text
STOP
```

and fix `/etc/fstab`.

Do not reboot until the configuration is valid.

A malformed `/etc/fstab` can cause boot or mounting problems.

---

# 🔄 Phase 14: Reboot and Validate Persistence

Reboot:

```bash
sudo reboot
```

Reconnect after the instance becomes available.

Run:

```bash
lsblk
```

Then:

```bash
df -h
```

Verify that:

```text
/mnt/data
```

is automatically mounted.

Finally:

```bash
cat /mnt/data/aws-data.txt
```

Expected:

```text
EBS Lab - Persistent Data
```

This proves two things:

```text
EBS Data
    │
    └── Survived Reboot

Mount Configuration
    │
    └── Survived Reboot
```

---

# 🧠 What I Learned from the Linux Exercise

There are actually two separate operations:

```text
AWS Layer
   │
   └── Attach EBS Volume

Linux Layer
   │
   ├── Detect Block Device
   ├── Create Filesystem
   ├── Create Mount Point
   └── Mount Filesystem
```

Attaching an EBS volume in AWS does not automatically make it usable as a Linux filesystem.

The operating system must still configure it.

---

# 🪟 Part 2: Windows EBS Data Volume

The same basic principle applies to Windows.

```text
AWS
 │
 └── Attach EBS Volume
          │
          ▼
Windows
 │
 ├── Detect Disk
 ├── Bring Online
 ├── Initialize
 ├── Create Volume
 ├── Format
 └── Assign Drive Letter
```

---

# 🖥️ Phase 15: Launch a Windows EC2 Instance

Launch a Windows EC2 instance for the lab.

Example:

```text
Name:
Windows-Server-01

AMI:
Supported Windows Server AMI

VPC:
Existing VPC

Subnet:
Public Subnet
```

Use a supported instance type appropriate for Windows.

Configure secure administrative access.

If RDP is used:

```text
TCP 3389
Source: My trusted IP
```

Do not expose RDP broadly unless there is a specific controlled requirement.

---

# 💾 Phase 16: Create the Windows Data Volume

Record the Windows instance Availability Zone.

Create another GP3 EBS volume.

Example:

```text
Name:
Windows-Data-Volume

Type:
gp3

Size:
30 GiB

Availability Zone:
Same as Windows-Server-01
```

Attach it to:

```text
Windows-Server-01
```

---

# 🪟 Phase 17: Make the Disk Available in Windows

Connect to the Windows instance.

Open:

```text
Disk Management
```

The newly attached disk should appear.

If required:

```text
Disk
 │
 ▼
Online
 │
 ▼
Initialize Disk
```

For a new empty volume, initialize it using the partition style appropriate for the workload.

Then:

```text
Unallocated Space
      │
      ▼
New Simple Volume
      │
      ▼
Assign Drive Letter
      │
      ▼
Format NTFS
```

For this lab:

```text
Drive Letter:
D:

Filesystem:
NTFS

Label:
Data
```

---

# ⚠️ Important: Existing Data Volumes

If the volume was created from a snapshot and already contains a filesystem:

```text
Do NOT initialize or format it again
```

Doing so could destroy the restored data.

This distinction becomes important later in the snapshot recovery exercise.

---

# ✍️ Phase 18: Create Windows Test Data

Open:

```text
D:\
```

Create:

```text
ebs-lab.txt
```

Add:

```text
EBS Snapshot Recovery Test
```

Save the file.

Our Windows storage now looks like:

```text
Windows EC2
│
├── C:
│   └── Root EBS
│
└── D:
    └── Data EBS
         │
         └── ebs-lab.txt
```

---

# 📸 Part 3: Create an EBS Snapshot

Now I want to protect the data independently of the EC2 instance.

The Windows data volume currently contains:

```text
D:\
   │
   └── ebs-lab.txt
```

Create a snapshot of:

```text
Windows-Data-Volume
```

Navigate to:

```text
EC2
 │
 ▼
Volumes
 │
 ▼
Windows-Data-Volume
 │
 ▼
Actions
 │
 ▼
Create Snapshot
```

Add:

```text
Name = Windows-Data-Snapshot
```

---

# 🧠 Application Consistency

For this simple lab text file, the workload is straightforward.

In production, creating a snapshot while an application is actively writing data requires more consideration.

A snapshot captures a point-in-time state of the EBS volume.

For application-consistent backups, applications such as databases may need their writes flushed or temporarily quiesced using the application's supported backup mechanism.

---

# ⏳ Phase 19: Monitor Snapshot Creation

The snapshot initially enters:

```text
pending
```

Eventually:

```text
completed
```

Snapshot creation is asynchronous.

The EBS volume itself can continue operating while the snapshot is being created.

---

# 🧠 Snapshot Architecture

Conceptually:

```text
EC2
 │
 ▼
EBS Volume
 │
 ▼
EBS Snapshot
```

An EBS snapshot is a point-in-time backup of an EBS volume.

The snapshot is a Regional resource.

It can be used to create EBS volumes in different Availability Zones within that Region.

---

# ♻️ Part 4: Restore an EBS Volume from a Snapshot

Once the snapshot is complete:

```text
Snapshot
   │
   ▼
Create Volume
```

Choose:

```text
Volume Type:
gp3

Availability Zone:
Same AZ as target EC2
```

The new volume can be the same size or larger than the source snapshot volume.

---

# 🔗 Phase 20: Attach the Restored Volume

Once the restored volume reaches:

```text
available
```

attach it to the target EC2 instance.

The important rule still applies:

```text
Restored EBS Volume AZ
          =
Target EC2 AZ
```

---

# ⚠️ Do Not Format a Restored Volume

This is an important troubleshooting and recovery lesson.

The restored volume already contains:

```text
Partition Table
       +
Filesystem
       +
Data
```

Therefore:

```text
Snapshot
   │
   ▼
Restored EBS
   │
   ▼
Attach
   │
   ▼
Mount / Bring Online

NOT

Format
```

Formatting the volume would destroy the filesystem containing the recovered data.

---

# ✅ Phase 21: Validate Recovered Data

For Windows, bring the restored disk online if necessary.

Open the restored data drive.

Verify:

```text
ebs-lab.txt
```

Open the file.

Expected:

```text
EBS Snapshot Recovery Test
```

We have now demonstrated:

```text
Original EBS
     │
     ▼
Snapshot
     │
     ▼
New EBS Volume
     │
     ▼
Recovered Data
```

---

# 🌍 Part 5: Cross-Region Snapshot Copy

Now I want to demonstrate how EBS snapshots can help move or recover data in another AWS Region.

Architecture:

```text
Region A
│
│  EBS Volume
│       │
│       ▼
│   Snapshot
│       │
└───────┼──────────────────┐
        │                  │
        │ Copy Snapshot    │
        ▼                  ▼
                         Region B
                            │
                            ▼
                     Copied Snapshot
                            │
                            ▼
                       EBS Volume
                            │
                            ▼
                         EC2
```

---

# 📤 Phase 22: Copy the Snapshot

Select:

```text
Windows-Data-Snapshot
```

Choose:

```text
Actions
   │
   ▼
Copy Snapshot
```

Select a destination Region.

For example:

```text
Source:
Canada Central

Destination:
US East (N. Virginia)
```

The actual Regions used in my lab can be different.

Wait until the copied snapshot becomes available in the destination Region.

---

# 🌎 Important: AWS Resources Are Region Scoped

When switching Regions, remember that many EC2 resources are Region-specific.

For example:

```text
EC2 Instances

EBS Volumes

EBS Snapshots

Security Groups

Key Pairs
```

do not automatically appear in another Region.

The destination Region therefore needs its own appropriate infrastructure.

---

# 🏗️ Phase 23: Prepare the Destination EC2 Instance

In the destination Region, create or use an EC2 instance.

Record its:

```text
Availability Zone
```

For example:

```text
us-east-1c
```

This determines where the restored EBS volume must be created.

---

# 💾 Phase 24: Create an EBS Volume from the Copied Snapshot

In the destination Region:

```text
Copied Snapshot
      │
      ▼
Create Volume
```

Choose:

```text
Volume Type:
gp3

Availability Zone:
Same AZ as destination EC2
```

Example:

```text
Destination EC2:
us-east-1c

Restored EBS:
us-east-1c
```

Not:

```text
Destination EC2:
us-east-1c

Restored EBS:
us-east-1a

❌
```

---

# 🔗 Phase 25: Attach the Restored Volume

Attach the newly created volume to the destination EC2 instance.

For Windows:

```text
Disk Management
      │
      ▼
Locate Restored Disk
      │
      ▼
Bring Online
```

Do not format it.

---

# 🔍 Phase 26: Validate Cross-Region Recovery

Open the restored volume.

Verify:

```text
ebs-lab.txt
```

Expected:

```text
EBS Snapshot Recovery Test
```

We have now demonstrated:

```text
Region A
EBS Volume
    │
    ▼
Snapshot
    │
    │ Cross-Region Copy
    ▼
Region B
Snapshot
    │
    ▼
EBS Volume
    │
    ▼
EC2
    │
    ▼
Original Data Recovered
```

---

# 🧠 What This Demonstrates

The original data was:

```text
Region A
    │
    ▼
EBS Volume
```

After using snapshots:

```text
Region A
    │
    ▼
Snapshot
    │
    ▼
Copy
    │
    ▼
Region B
    │
    ▼
New EBS Volume
```

This capability can form part of:

```text
Disaster Recovery

Data Migration

Environment Duplication

Testing

Backup and Recovery
```

However, a snapshot copy alone is not a complete disaster recovery architecture.

Networking, compute, configuration, security, applications, databases, DNS, and recovery procedures may also need to be recreated or automated.

---

# 🔍 Troubleshooting

## Problem 1: EC2 Instance Does Not Appear When Attaching the Volume

Check:

```text
EBS Availability Zone
        │
        ▼
EC2 Availability Zone
```

They must match.

---

## Problem 2: Linux Does Not Show the Expected Device Name

Run:

```bash
lsblk
```

Do not assume:

```text
/dev/sdf
```

in the EC2 console will necessarily appear under that exact name inside Linux.

Nitro instances commonly expose EBS devices as NVMe devices.

---

## Problem 3: Volume is Attached but `df -h` Does Not Show It

Remember:

```text
Attach
   ≠
Mount
```

Check:

```bash
lsblk
```

Then determine whether the volume needs:

```text
Filesystem
     +
Mount Point
     +
Mount
```

---

## Problem 4: Volume Does Not Mount After Reboot

Check:

```bash
cat /etc/fstab
```

Then:

```bash
sudo mount -a
```

Verify:

```text
UUID

Mount Point

Filesystem Type

fstab Syntax
```

---

## Problem 5: Restored Snapshot Data Disappeared

Ask:

```text
Did I format the restored volume?
```

A volume created from a snapshot already contains the source data.

Do not run:

```bash
mkfs
```

on a restored volume containing data you want to preserve.

---

## Problem 6: Cannot Attach Restored Volume to Destination EC2

Check:

```text
Destination Region

Destination Availability Zone

EC2 Availability Zone

EBS Availability Zone
```

The EC2 instance and EBS volume must be in the same Availability Zone.

---

# 🧪 Break/Fix Exercise 1: Wrong Availability Zone

Create or attempt to create an EBS volume in:

```text
AZ-B
```

while the EC2 instance exists in:

```text
AZ-A
```

Try to attach it.

Observe what happens.

Then correct the design:

```text
EC2
AZ-A

EBS
AZ-A
```

### Lesson

```text
EBS Volumes are
Availability Zone scoped
```

---

# 🧪 Break/Fix Exercise 2: Remove the Persistent Mount

On Linux:

```bash
sudo cp /etc/fstab /etc/fstab.lab-backup
```

Temporarily remove or comment out the `/mnt/data` entry.

Reboot.

Observe:

```text
EBS Attached
     │
     ▼
Device Exists
     │
     ▼
Filesystem Exists
     │
     ▼
Not Automatically Mounted
```

Restore the correct `/etc/fstab` configuration afterward.

### Lesson

AWS attachment and Linux mounting are separate layers.

---

# 🧪 Break/Fix Exercise 3: Snapshot Recovery

After creating the snapshot:

1. Change or remove the test file on the original volume.
2. Create a new EBS volume from the earlier snapshot.
3. Attach the restored volume.
4. Verify the earlier version of the file.

### Lesson

A snapshot represents the volume's state at a specific point in time.

---

# 📊 Quick Reference

| Task                          | AWS Layer | OS Layer |
| ----------------------------- | --------- | -------- |
| Create EBS volume             | ✅         |          |
| Attach EBS volume             | ✅         |          |
| Detect block device           |           | ✅        |
| Create filesystem             |           | ✅        |
| Mount filesystem              |           | ✅        |
| Configure persistent mount    |           | ✅        |
| Initialize Windows disk       |           | ✅        |
| Format NTFS                   |           | ✅        |
| Create snapshot               | ✅         |          |
| Restore volume                | ✅         |          |
| Copy snapshot between Regions | ✅         |          |
| Validate restored files       |           | ✅        |

---

# ❓ Interview Questions

### Q1. Can an EBS volume in one Availability Zone be attached to an EC2 instance in another Availability Zone?

**Answer**

No.

An EBS volume must be in the same Availability Zone as the EC2 instance to which it is attached.

---

### Q2. What does the `available` state of an EBS volume mean?

**Answer**

The volume exists but is not currently attached to an EC2 instance.

---

### Q3. What does `in-use` mean?

**Answer**

The EBS volume is currently attached to an EC2 instance.

---

### Q4. After attaching an empty EBS volume to Linux, can applications immediately store files on it?

**Answer**

Not normally.

The operating system must identify the block device, create an appropriate filesystem, and mount that filesystem before applications can use it as normal file storage.

---

### Q5. What does `lsblk` do?

**Answer**

It displays information about Linux block devices and helps identify attached storage devices and their mount points.

---

### Q6. Why use a filesystem UUID in `/etc/fstab`?

**Answer**

Device names can change. A filesystem UUID provides a more stable way to identify the filesystem that should be mounted.

---

### Q7. What is `/etc/fstab`?

**Answer**

It is a Linux configuration file that defines filesystems that should be mounted, including where and with which options.

---

### Q8. Why run `mount -a` before rebooting?

**Answer**

It provides a way to test the `/etc/fstab` entries and identify configuration errors before rebooting the instance.

---

### Q9. What happens when a new EBS volume is attached to Windows?

**Answer**

Windows detects the disk, but a new empty disk might need to be brought online, initialized, partitioned, formatted, and assigned a drive letter before it can be used.

---

### Q10. What is an EBS snapshot?

**Answer**

An EBS snapshot is a point-in-time backup of an EBS volume.

---

### Q11. Can I create an EBS volume in another Availability Zone from a snapshot?

**Answer**

Yes.

Because the snapshot is Regional, I can create a new EBS volume from it in an Availability Zone in that Region, subject to AWS-supported locations and configurations.

---

### Q12. How do I move EBS data to another Region?

**Answer**

A common approach is:

```text
EBS Volume
    │
    ▼
Snapshot
    │
    ▼
Copy Snapshot
to Destination Region
    │
    ▼
Create EBS Volume
    │
    ▼
Attach to EC2
```

---

### Q13. Should I format a volume restored from a snapshot?

**Answer**

No, not when I want to preserve the snapshot's existing filesystem and data.

Formatting would overwrite the existing filesystem.

---

### Q14. Can the restored EBS volume be larger than the original snapshot volume?

**Answer**

Yes.

A volume created from a snapshot can be created at the snapshot size or larger.

The operating system filesystem might then need to be extended to use the additional capacity.

---

### Q15. Does creating a snapshot require stopping the EC2 instance?

**Answer**

Not necessarily.

AWS allows snapshots to be created while a volume is in use. However, for workloads with active writes, additional steps may be required to achieve application-consistent backups.

---

### Q16. Why might a volume restored from a snapshot initially have higher I/O latency?

**Answer**

Blocks from the snapshot are loaded to the new volume as they are accessed. Until the necessary blocks are initialized, the volume can experience increased first-access latency.

---

### Q17. What is the difference between an EBS volume and an EBS snapshot?

**Answer**

```text
EBS Volume
    │
    └── Active block storage
        attached to EC2


EBS Snapshot
    │
    └── Point-in-time backup
        used to create volumes
```

---

# 💰 Cost Awareness

This lab can create billable resources:

```text
EC2 Instances

EBS Volumes

EBS Snapshots

Public IPv4 Addresses

Cross-Region Snapshot Copy

Data Transfer
```

Do not assume that an old course's Free Tier limits or pricing still apply.

Always check current AWS pricing and the account's actual Free Tier or credit eligibility before running the lab.

---

# 🧹 Cleanup

After completing the lab:

```text
Terminate Linux EC2

Terminate Windows EC2

Terminate Destination EC2

Delete Unneeded Data Volumes

Delete Restored Volumes

Delete Source Snapshot

Delete Cross-Region Snapshot Copy
```

After terminating EC2 instances, check:

```text
EC2
  │
  ▼
Volumes
```

Do not assume every EBS volume was automatically deleted.

Root volumes are commonly configured with:

```text
DeleteOnTermination = true
```

while separately attached data volumes can remain after the instance is terminated.

Verify rather than assume.

Also check every Region used during the lab.

---

# ✅ Lab Completion Criteria

I consider this lab complete when I can demonstrate, without blindly following the instructions:

```text
☐ Create a GP3 EBS volume

☐ Explain why the EBS and EC2 AZ must match

☐ Attach the volume to Linux

☐ Identify the correct Linux block device

☐ Create an EXT4 filesystem

☐ Mount the volume

☐ Store test data

☐ Configure /etc/fstab using UUID

☐ Reboot and prove the mount persists

☐ Attach a data volume to Windows

☐ Bring the Windows disk online

☐ Initialize and format a new Windows disk

☐ Store test data

☐ Create an EBS snapshot

☐ Restore a new EBS volume from the snapshot

☐ Recover the original file

☐ Copy a snapshot to another Region

☐ Create an EBS volume in the destination Region

☐ Explain why the destination EBS volume must match the target EC2 AZ

☐ Recover the data in another Region

☐ Troubleshoot an incorrect AZ

☐ Explain the difference between attach, format, and mount

☐ Clean up all resources
```

---

# 💡 Key Takeaways

* Amazon EBS provides persistent block storage for EC2.
* An EBS volume must be in the same Availability Zone as the EC2 instance to which it is attached.
* Attaching an EBS volume and configuring it inside the operating system are separate operations.
* Linux needs a filesystem and mount point before a new empty EBS volume can be used for normal file storage.
* `/etc/fstab` can make Linux filesystem mounts persistent across reboots.
* UUIDs provide a reliable way to identify filesystems in `/etc/fstab`.
* Windows may require a new empty EBS disk to be brought online, initialized, formatted, and assigned a drive letter.
* Never format a restored volume containing data that needs to be preserved.
* EBS snapshots provide point-in-time backups of EBS volumes.
* A snapshot can be used to create new volumes in different Availability Zones within its Region.
* A snapshot can be copied to another Region and used to create EBS volumes there.
* A volume restored from a snapshot can be larger than the original, although the filesystem may need to be expanded.
* EBS snapshots can support backup, migration, testing, and disaster recovery strategies.
* Snapshot recovery demonstrates an important cloud principle: **the data can survive independently of the original EC2 instance.**

---

# 📚 Related Topics

* Amazon EBS
* EBS Volume Types
* GP2 and GP3
* Provisioned IOPS
* EC2 Instance Store
* EBS Snapshots
* EBS Encryption
* EBS Volume Modification
* Linux Filesystems
* Linux `/etc/fstab`
* EC2 Nitro System
* AWS Backup
* Amazon Data Lifecycle Manager
* Disaster Recovery

---

# 📖 References

* AWS Documentation: Amazon EBS Volumes
* AWS Documentation: Create an Amazon EBS Volume
* AWS Documentation: Attach an Amazon EBS Volume to an Instance
* AWS Documentation: Make an Amazon EBS Volume Available for Use
* AWS Documentation: Create Amazon EBS Snapshots
* AWS Documentation: Restore an Amazon EBS Volume from a Snapshot
* AWS Documentation: Copy Amazon EBS Snapshots
* AWS Documentation: Amazon EBS Volume Lifecycle
