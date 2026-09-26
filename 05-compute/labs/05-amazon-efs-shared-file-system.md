# 🧪 Lab 01: Build a Shared File System with Amazon EFS

> Build a Regional Amazon EFS filesystem, connect two Linux EC2 instances from different Availability Zones, and verify that both servers can access and modify the same files.

---

# 🎯 Lab Objective

The goal of this lab is to understand Amazon EFS by actually building a shared filesystem.

By the end of the lab, I should be able to explain:

* Why EFS is different from EBS
* How EFS mount targets work
* Why mount targets are created in different Availability Zones
* How security groups protect EFS
* Why NFS uses TCP port 2049
* How Linux mounts an EFS filesystem
* How multiple EC2 instances access the same files
* How DNS is used to access EFS
* Why Regional EFS supports highly available applications
* How to perform basic EFS operations using the AWS CLI

---

# 🏗️ Architecture

I will build:

```text
                       AWS Region
                           │
                           ▼
                         VPC
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
            AZ-A                        AZ-B
             │                           │
             ▼                           ▼
        Public Subnet               Public Subnet
             │                           │
             ▼                           ▼
          EC2-A                       EC2-B
        AppServer-A                 AppServer-B
             │                           │
             │ NFS 2049                  │ NFS 2049
             ▼                           ▼
       EFS Mount Target            EFS Mount Target
             │                           │
             └─────────────┬─────────────┘
                           │
                           ▼
                    Regional EFS
                           │
                           ▼
                      Shared Files
```

The important part is:

```text
EC2-A ──────┐
            │
            ▼
        Amazon EFS
            ▲
            │
EC2-B ──────┘
```

Both servers will access the **same filesystem**.

---

# 🧠 What I Am Proving

Before building anything, I want to be clear about what this lab demonstrates.

If I use EBS:

```text
EC2-A ──► EBS-A

EC2-B ──► EBS-B
```

each server normally has its own block storage.

With EFS:

```text
EC2-A ──┐
        ├──► Shared EFS
EC2-B ──┘
```

both servers can see the same files.

I will prove this by:

```text
EC2-A
  │
  ▼
Create hello.txt
  │
  ▼
Amazon EFS
  │
  ▼
Read hello.txt
  │
  ▼
EC2-B
```

Then I will modify the file from EC2-B and verify the change from EC2-A.

---

# 📋 Resources Required

I will create:

| Resource                |         Quantity |
| ----------------------- | ---------------: |
| VPC                     | Existing/default |
| Public Subnets          |                2 |
| Availability Zones      |                2 |
| EC2 Linux Instances     |                2 |
| EC2 Security Group      |                1 |
| EFS Security Group      |                1 |
| Regional EFS Filesystem |                1 |
| EFS Mount Targets       |                2 |

---

# 🏷️ Suggested Names

Use simple names so the architecture is easy to understand.

```text
EC2 Instances

efs-app-a
efs-app-b


Security Groups

efs-lab-ec2-sg
efs-lab-efs-sg


EFS

efs-lab-shared
```

---

# 💰 Cost Awareness

This lab can incur charges for resources such as:

* EC2 instances
* Amazon EFS storage
* Data transfer in some scenarios

Keep the lab small and clean up everything when finished.

Do not leave resources running just because the lab is complete.

---

# Phase 1: Understand the Network

For this lab I need two subnets in different Availability Zones.

Example:

```text
VPC
│
├── AZ-A
│    └── Public Subnet A
│
└── AZ-B
     └── Public Subnet B
```

The original exercise uses the default VPC, so I can use that for this isolated learning lab.

Before continuing, identify:

```text
VPC ID

Subnet A ID
Availability Zone A

Subnet B ID
Availability Zone B
```

Example only:

```text
VPC:
vpc-xxxxxxxx

Subnet A:
subnet-aaaaaaaa
AZ: us-east-1a

Subnet B:
subnet-bbbbbbbb
AZ: us-east-1b
```

Do not copy these example IDs.

Use the IDs from my own AWS account.

---

# Phase 2: Create the EC2 Security Group

Create:

```text
efs-lab-ec2-sg
```

Attach it to the VPC being used for the lab.

For browser-based EC2 Instance Connect, configure access according to the connection method and network architecture I choose.

If I deliberately use direct SSH from my computer, restrict SSH to:

```text
TCP 22
Source: My IP
```

Avoid:

```text
SSH
TCP 22
0.0.0.0/0
```

unless there is a specific temporary lab reason and I understand the exposure.

The important EFS security rule will come later.

---

# Phase 3: Launch Two EC2 Instances

Launch two small Amazon Linux instances.

Place them in different Availability Zones.

```text
efs-app-a
    │
    ▼
Subnet A
    │
    ▼
AZ-A


efs-app-b
    │
    ▼
Subnet B
    │
    ▼
AZ-B
```

Attach:

```text
efs-lab-ec2-sg
```

to both instances.

Use small instance types suitable for a short learning lab.

---

# ✅ Validation Checkpoint 1

Before continuing, verify:

```text
EC2-A → Running

EC2-B → Running

Different AZs → Yes

Same VPC → Yes

Correct Security Group → Yes
```

I should be able to explain why the instances are in different Availability Zones:

> I want to demonstrate how a Regional EFS filesystem can provide shared file access to workloads running across multiple Availability Zones.

---

# Phase 4: Connect to the Instances

Connect to both instances using the connection method selected for the lab.

For example:

```text
EC2 Instance Connect
```

Open one terminal for:

```text
efs-app-a
```

and another for:

```text
efs-app-b
```

Keeping both terminals open will make the shared-filesystem test easier.

---

# Phase 5: Install NFS Utilities

Amazon EFS uses NFS.

On both EC2 instances, install the NFS client utilities.

For Amazon Linux:

```bash
sudo dnf install -y nfs-utils
```

Depending on the Amazon Linux version, `yum` may also be available.

Verify:

```bash
rpm -q nfs-utils
```

I should understand why this package is required:

```text
EC2
 │
 ▼
NFS Client
 │
 ▼
EFS
```

Without an NFS client, Linux cannot mount the EFS filesystem using the NFS method used in this lab.

---

# Phase 6: Open AWS CloudShell

For the infrastructure portion of the exercise, use AWS CloudShell.

First verify who I am:

```bash
aws sts get-caller-identity
```

Then verify the Region:

```bash
aws configure get region
```

If necessary, set the Region used for the lab.

For example:

```bash
export AWS_REGION=us-east-1
```

Replace the Region with the Region I am actually using.

---

# Phase 7: Record Resource IDs

Create variables to make the commands easier to understand.

```bash
VPC_ID="vpc-xxxxxxxx"
EC2_SG_ID="sg-xxxxxxxx"
SUBNET_A="subnet-xxxxxxxx"
SUBNET_B="subnet-yyyyyyyy"
```

Replace every placeholder with my actual resource IDs.

Verify:

```bash
echo $VPC_ID
echo $EC2_SG_ID
echo $SUBNET_A
echo $SUBNET_B
```

---

# Phase 8: Create the EFS Security Group

The EFS mount targets need their own security group.

Create it:

```bash
EFS_SG_ID=$(aws ec2 create-security-group \
  --group-name efs-lab-efs-sg \
  --description "Security group for EFS mount targets" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' \
  --output text)
```

Check:

```bash
echo $EFS_SG_ID
```

---

# 🧠 Why Use Two Security Groups?

The architecture now becomes:

```text
EC2
 │
 │ efs-lab-ec2-sg
 ▼

Network

 ▼
EFS Mount Target
 │
 │ efs-lab-efs-sg
 ▼
EFS
```

Instead of trusting an entire CIDR range, I can say:

> Allow NFS connections to EFS only from resources associated with my EC2 application security group.

That gives me a cleaner security relationship.

---

# Phase 9: Allow NFS Traffic

EFS uses:

```text
TCP 2049
```

Configure the EFS security group:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$EFS_SG_ID" \
  --protocol tcp \
  --port 2049 \
  --source-group "$EC2_SG_ID"
```

The resulting rule should conceptually be:

```text
efs-lab-efs-sg

Inbound:

Type: NFS
Protocol: TCP
Port: 2049
Source: efs-lab-ec2-sg
```

Architecture:

```text
EC2
 │
 │ TCP 2049
 ▼
EFS Security Group
 │
 ▼
Mount Target
```

---

# ✅ Validation Checkpoint 2

Open the EFS security group in the console.

Verify:

```text
Protocol: TCP

Port: 2049

Source:
EC2 Security Group
```

The source should **not** unnecessarily be:

```text
0.0.0.0/0
```

---

# Phase 10: Create the EFS Filesystem

Now create an encrypted Regional EFS filesystem.

```bash
EFS_ID=$(aws efs create-file-system \
  --creation-token efs-lab-shared \
  --encrypted \
  --tags Key=Name,Value=efs-lab-shared \
  --query 'FileSystemId' \
  --output text)
```

Check:

```bash
echo $EFS_ID
```

Then inspect it:

```bash
aws efs describe-file-systems \
  --file-system-id "$EFS_ID"
```

Wait until:

```text
LifeCycleState = available
```

---

# 🧠 What Did I Just Create?

At this point:

```text
Regional EFS
```

exists.

But my EC2 instances still need network endpoints through which they can reach it.

Those endpoints are:

```text
Mount Targets
```

So currently:

```text
EC2-A

EC2-B


       EFS
```

The next step creates the network path.

---

# Phase 11: Create Mount Target in AZ-A

Create a mount target in the subnet containing EC2-A:

```bash
aws efs create-mount-target \
  --file-system-id "$EFS_ID" \
  --subnet-id "$SUBNET_A" \
  --security-groups "$EFS_SG_ID"
```

---

# Phase 12: Create Mount Target in AZ-B

Create the second mount target:

```bash
aws efs create-mount-target \
  --file-system-id "$EFS_ID" \
  --subnet-id "$SUBNET_B" \
  --security-groups "$EFS_SG_ID"
```

Now:

```text
             Regional EFS
                  │
          ┌───────┴───────┐
          │               │
          ▼               ▼
     Mount Target     Mount Target
          │               │
         AZ-A             AZ-B
```

---

# Phase 13: Verify Mount Targets

Run:

```bash
aws efs describe-mount-targets \
  --file-system-id "$EFS_ID"
```

I should eventually see two mount targets.

Wait until both show:

```text
LifeCycleState = available
```

Also verify in the EFS console:

```text
EFS
 │
 └── Network
      │
      ├── AZ-A → Mount Target
      └── AZ-B → Mount Target
```

---

# 🧠 Important Concept

I am not creating:

```text
EFS-A

EFS-B
```

I created:

```text
ONE Regional EFS Filesystem
```

with:

```text
TWO Mount Targets
```

The mount targets provide access to the same filesystem.

---

# Phase 14: Determine the EFS DNS Name

The filesystem DNS name follows the EFS DNS naming format for the Region.

The easiest approach is to use the **Attach** option in the EFS console and copy the provided mount command.

Conceptually:

```text
EC2
 │
 ▼
EFS DNS Name
 │
 ▼
Local-AZ Mount Target
 │
 ▼
EFS
```

---

# Phase 15: Mount EFS on EC2-A

Return to:

```text
efs-app-a
```

Create a mount directory:

```bash
sudo mkdir -p /mnt/efs
```

Mount the filesystem using NFS.

Use the exact DNS name shown by the EFS console:

```bash
sudo mount -t nfs4 \
  -o nfsvers=4.1 \
  <EFS-DNS-NAME>:/ \
  /mnt/efs
```

Verify:

```bash
df -hT
```

and:

```bash
mount | grep efs
```

---

# Phase 16: Create Shared Data from EC2-A

For this short lab, give the EC2 user ownership of the mount directory:

```bash
sudo chown ec2-user:ec2-user /mnt/efs
```

Move into it:

```bash
cd /mnt/efs
```

Create a file:

```bash
echo "Hello from EC2-A" > hello.txt
```

Verify:

```bash
cat hello.txt
```

Expected:

```text
Hello from EC2-A
```

List it:

```bash
ls -l
```

---

# Phase 17: Mount EFS on EC2-B

Now connect to:

```text
efs-app-b
```

Create the directory:

```bash
sudo mkdir -p /mnt/efs
```

Mount the **same EFS filesystem**:

```bash
sudo mount -t nfs4 \
  -o nfsvers=4.1 \
  <EFS-DNS-NAME>:/ \
  /mnt/efs
```

Verify:

```bash
df -hT
```

Then:

```bash
cd /mnt/efs
ls -l
```

I should see:

```text
hello.txt
```

without creating it on EC2-B.

Read it:

```bash
cat hello.txt
```

Expected:

```text
Hello from EC2-A
```

---

# 🎯 Major Validation

This is the most important moment in the lab.

I created:

```text
hello.txt
```

on:

```text
EC2-A
```

but I can read it from:

```text
EC2-B
```

Why?

Because the file is **not stored locally on EC2-A**.

It is stored on:

```text
Amazon EFS
```

Both servers mounted the same filesystem.

---

# Phase 18: Modify the File from EC2-B

On EC2-B:

```bash
echo "Updated from EC2-B" >> /mnt/efs/hello.txt
```

Check:

```bash
cat /mnt/efs/hello.txt
```

Expected:

```text
Hello from EC2-A
Updated from EC2-B
```

---

# Phase 19: Verify the Change from EC2-A

Return to EC2-A:

```bash
cat /mnt/efs/hello.txt
```

Expected:

```text
Hello from EC2-A
Updated from EC2-B
```

This proves:

```text
EC2-A
   │
   │
   ▼
Shared File
   ▲
   │
   │
EC2-B
```

Both servers are working with the same shared filesystem.

---

# 🔬 Phase 20: Inspect the Filesystem

Run on either instance:

```bash
df -hT
```

Then:

```bash
mount
```

and:

```bash
ls -la /mnt/efs
```

This helps connect the AWS architecture to what Linux actually sees.

From Linux's perspective:

```text
/mnt/efs
```

looks like another mounted filesystem.

But behind it:

```text
Linux
 │
 ▼
NFS
 │
 ▼
Network
 │
 ▼
EFS Mount Target
 │
 ▼
Regional EFS
```

---

# 💥 Break/Fix Exercise 1: Block NFS

Now deliberately break the architecture.

Remove the inbound NFS rule from:

```text
efs-lab-efs-sg
```

Then unmount EFS from one server:

```bash
sudo umount /mnt/efs
```

Try mounting it again.

What happens?

The mount should fail or time out because:

```text
EC2
 │
 │ TCP 2049
 ▼
EFS SG
 │
 X BLOCKED
```

---

# 🔧 Fix

Restore:

```text
Inbound

TCP 2049

Source:
efs-lab-ec2-sg
```

Try mounting again.

The mount should succeed.

---

# 🧠 Troubleshooting Lesson

If EFS cannot be mounted, I should not immediately assume:

```text
EFS is broken
```

I should troubleshoot layer by layer:

```text
1. Is EFS available?

2. Is the mount target available?

3. Is there a mount target in the expected AZ?

4. Is DNS resolution working?

5. Does the EFS security group allow TCP 2049?

6. Is the correct EC2 security group the source?

7. Does the EC2 outbound configuration allow the traffic?

8. Is the NFS client installed?

9. Is the mount command correct?
```

---

# 💥 Break/Fix Exercise 2: Wrong Security Group Source

Change the EFS rule so that the source no longer references:

```text
efs-lab-ec2-sg
```

Unmount and attempt the connection again.

This demonstrates that security-group references are not just labels.

They actually define:

```text
Who Can Connect?
```

Restore the correct source afterward.

---

# ⭐ Optional Challenge: Prove the Data Is Not Local

On EC2-A:

```bash
sudo umount /mnt/efs
```

Now:

```bash
ls /mnt/efs
```

The shared file should no longer appear through that mount point.

Mount EFS again:

```bash
sudo mount -t nfs4 \
  -o nfsvers=4.1 \
  <EFS-DNS-NAME>:/ \
  /mnt/efs
```

Then:

```bash
cat /mnt/efs/hello.txt
```

The data appears again.

This demonstrates:

```text
File != Local EC2 Disk

File = Stored on EFS
```

---

# ⭐ Optional Challenge: Persistent Mount

The manual `mount` command does not automatically guarantee that the filesystem will remain mounted after a reboot.

For a production-style configuration, I would configure persistent mounting using:

```text
/etc/fstab
```

or use the Amazon EFS mount helper.

I will treat this as a separate Linux/EFS exercise rather than blindly editing `/etc/fstab` in this first lab.

---

# 🆚 What This Lab Taught Me About EBS vs EFS

## EBS Architecture

```text
EC2-A ──► EBS-A


EC2-B ──► EBS-B
```

Storage is primarily attached as block devices.

---

## EFS Architecture

```text
EC2-A ──┐
        │
        ▼
       EFS
        ▲
        │
EC2-B ──┘
```

Storage is accessed as a shared network filesystem.

---

# ❓ Interview Questions

### Q1. Why did we use EFS instead of EBS?

Because two EC2 instances needed concurrent access to the same shared files.

---

### Q2. Why did we create two mount targets?

The EC2 instances were running in different Availability Zones.

For a Regional EFS filesystem, creating a mount target in each AZ where clients run provides local access to the filesystem.

---

### Q3. Did we create two EFS filesystems?

No.

We created:

```text
1 Regional EFS Filesystem

2 Mount Targets
```

---

### Q4. What port does EFS use?

```text
TCP 2049
```

for NFS.

---

### Q5. Why did the EFS security group reference the EC2 security group?

It allows NFS traffic from the intended application instances without broadly allowing an entire internet or network range.

---

### Q6. What protocol did the EC2 instances use to access EFS?

```text
NFS
```

---

### Q7. Why did we install `nfs-utils`?

The Linux instances needed an NFS client to mount the EFS filesystem using NFS.

---

### Q8. Where was `hello.txt` actually stored?

On the EFS filesystem.

It was not stored independently on EC2-A or EC2-B.

---

### Q9. How did EC2-B see a file created by EC2-A?

Both EC2 instances mounted the same EFS filesystem.

---

### Q10. What happens if TCP 2049 is blocked?

The EC2 instance cannot establish the required NFS connection to the EFS mount target.

---

### Q11. Is a mount target the filesystem itself?

No.

The mount target provides a network endpoint through which clients access the filesystem.

---

### Q12. Why use mount targets in multiple AZs?

It allows workloads in different Availability Zones to access the Regional EFS filesystem through mount targets in their respective AZs.

---

### Q13. Does EFS require me to choose a 100 GiB or 500 GiB filesystem size?

No.

EFS capacity automatically grows and shrinks as files are added and removed.

---

### Q14. Could I use EFS for the EC2 root operating system disk?

That is not the normal purpose of EFS.

EC2 root volumes typically use EBS.

EFS is designed for shared file-storage use cases.

---

### Q15. What would I troubleshoot first if EFS would not mount?

I would check:

```text
EFS status
      ↓
Mount target
      ↓
Networking
      ↓
Security groups
      ↓
TCP 2049
      ↓
DNS
      ↓
NFS client
      ↓
Mount command
```

---

# 🧹 Cleanup

Do not skip cleanup.

## 1. Unmount EFS

On both EC2 instances:

```bash
sudo umount /mnt/efs
```

---

## 2. Terminate EC2 Instances

Terminate:

```text
efs-app-a

efs-app-b
```

Wait until termination progresses before deleting dependent security groups.

---

## 3. Delete EFS

Delete:

```text
efs-lab-shared
```

Deleting the filesystem permanently deletes the data stored in it.

Make sure no important data exists before confirming deletion.

---

## 4. Delete Security Groups

Delete:

```text
efs-lab-efs-sg

efs-lab-ec2-sg
```

If AWS reports that a security group is still in use, wait for the dependent network interfaces or instances to be removed and try again.

---

## 5. Keep the Default VPC

If I used the existing default VPC and its default subnets:

```text
DO NOT DELETE THEM
```

They were not created specifically for this lab.

---

# ✅ Completion Criteria

I consider this lab complete only when I can do all of the following:

* [ ] Explain EBS vs EFS
* [ ] Launch EC2 instances in two AZs
* [ ] Create an EFS filesystem
* [ ] Explain what an EFS mount target is
* [ ] Create mount targets in two AZs
* [ ] Configure TCP 2049 correctly
* [ ] Use a security group as another security group's source
* [ ] Install an NFS client on Linux
* [ ] Mount EFS on EC2-A
* [ ] Mount the same EFS on EC2-B
* [ ] Create a file from EC2-A
* [ ] Read that file from EC2-B
* [ ] Modify the file from EC2-B
* [ ] See the modification from EC2-A
* [ ] Break NFS connectivity intentionally
* [ ] Diagnose why the mount fails
* [ ] Restore connectivity
* [ ] Explain why the data is stored on EFS rather than locally
* [ ] Clean up all lab resources

---

# 💡 Key Takeaways

* EFS provides shared file storage.
* EBS and EFS solve different storage problems.
* A Regional EFS filesystem can support clients across multiple Availability Zones.
* Mount targets provide network access to EFS.
* Regional EFS supports one mount target per Availability Zone.
* EFS uses NFS.
* NFS traffic to EFS uses TCP port 2049.
* Security groups control access to EFS mount targets.
* Security-group references provide a clean way to allow application servers to access EFS.
* Multiple EC2 instances can mount the same EFS filesystem.
* Files written by one instance can immediately be available to another instance through the shared filesystem.
* EFS capacity does not need to be preallocated like an EBS volume.
* Understanding Linux mounting is important for working with EFS.
* Troubleshooting EFS requires understanding networking, security groups, DNS, NFS, and Linux mounts.
* Cleaning up resources is part of completing the lab.

---

# 📚 Related Topics

* Amazon EFS
* Amazon EBS
* NFS
* Linux Mount Points
* `/etc/fstab`
* Amazon EFS Mount Helper
* EFS Access Points
* Security Groups
* EFS Encryption
* EFS Lifecycle Management
* EFS Storage Classes
* EFS Throughput Modes
* Multi-AZ Architecture
* AWS CloudShell
* AWS CLI
