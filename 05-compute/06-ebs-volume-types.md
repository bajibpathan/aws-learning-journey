# 💾 Amazon EBS Volume Types: General Purpose and Provisioned IOPS SSD

> Amazon EBS provides different volume types so that storage performance and cost can be matched to the requirements of the workload.

---

# 📖 Overview

After learning how Amazon EBS provides persistent block storage for EC2, the next thing I needed to understand was:

> Which EBS volume type should I use?

Not every application has the same storage requirements.

For example:

```text
Web Server
    │
    └── General storage requirements

Application Server
    │
    └── Balanced price and performance

Database
    │
    └── Consistent IOPS may be important

Mission-Critical Database
    │
    └── Very high IOPS + low latency
```

For SSD-backed EBS storage, the main options are:

```text
SSD-backed EBS
│
├── General Purpose SSD
│     ├── gp3
│     └── gp2
│
└── Provisioned IOPS SSD
      ├── io2 Block Express
      └── io1
```

The key lesson for me is that selecting storage should start with the **application's I/O requirements**, not simply with the amount of disk space required.

---

# 🧠 Understanding Storage Performance

Before comparing EBS volume types, I need to understand three important concepts:

```text
IOPS
   +
Throughput
   +
Latency
```

They measure different aspects of storage performance.

---

# ⚡ What are IOPS?

IOPS means:

```text
Input / Output
Operations
Per Second
```

It measures how many read or write operations a storage system can perform each second.

For example:

```text
Application
    │
    ├── Read
    ├── Write
    ├── Read
    ├── Write
    └── Read

       ↓

Number of operations
completed per second

       ↓

IOPS
```

High IOPS are especially important for workloads performing many small, frequent storage operations.

Examples include:

* Transactional databases
* Database indexes
* High-volume application transactions

---

# 🚚 What is Throughput?

Throughput measures how much data can be transferred over time.

For EBS it is commonly expressed as:

```text
MiB/s
```

The easiest way for me to remember the difference is:

```text
IOPS
 │
 └── How many operations?


Throughput
 │
 └── How much data?


Latency
 │
 └── How long does an operation take?
```

Different workloads care about these characteristics differently.

---

# 💿 General Purpose SSD

General Purpose SSD volumes are designed to provide a balance between:

```text
Performance
    +
Cost
```

AWS currently provides two General Purpose SSD volume types:

```text
gp3

gp2
```

Both are SSD-backed and can be used as boot volumes.

General Purpose SSD is suitable for many common workloads such as:

* Virtual desktops
* Development and test environments
* Interactive applications
* Medium-sized single-instance databases
* General application workloads

---

# 🟢 GP3: General Purpose SSD

GP3 is the current-generation General Purpose SSD volume.

The most important thing I learned about GP3 is that:

> Storage capacity and storage performance can be configured independently.

For example:

```text
GP3 Volume
│
├── Capacity
│
├── IOPS
│
└── Throughput
```

Increasing IOPS does not necessarily mean increasing the size of the disk.

This provides much more flexibility when right-sizing storage.

---

# ⚙️ GP3 Baseline Performance

Every GP3 volume includes baseline performance of:

```text
3,000 IOPS

+

125 MiB/s Throughput
```

without requiring additional provisioned performance.

Another important characteristic is:

```text
GP3
 │
 └── No I/O credit system
```

GP3 does not depend on burst credits to maintain its provisioned performance.

---

# 🚀 Scaling GP3 Performance

If the baseline performance is not enough, additional performance can be provisioned independently.

Current GP3 limits include:

```text
IOPS
3,000
   │
   ▼
Up to 80,000


Throughput
125 MiB/s
   │
   ▼
Up to 2,000 MiB/s
```

Additional IOPS and throughput above the included baseline incur additional charges.

---

# 📏 GP3 Capacity

GP3 volumes currently support sizes from:

```text
1 GiB
  │
  ▼
64 TiB
```

This is another area where current AWS capabilities have expanded beyond some older training material.

---

# 🧮 GP3 IOPS-to-Size Relationship

GP3 supports provisioning performance independently of storage size, but AWS still applies maximum ratios.

For IOPS:

```text
Maximum Ratio

500 IOPS
per
GiB
```

For example:

```text
160 GiB GP3
      │
      │ × 500
      ▼
80,000 IOPS
```

Therefore, a GP3 volume must be at least 160 GiB to provision the current maximum of 80,000 IOPS.

---

# 🚚 GP3 Throughput Scaling

GP3 includes:

```text
125 MiB/s
```

of baseline throughput.

Additional throughput can be provisioned up to:

```text
2,000 MiB/s
```

AWS applies a relationship of:

```text
0.25 MiB/s throughput
per provisioned IOPS
```

This means that IOPS and throughput still need to be considered together when configuring higher-performance GP3 volumes.

---

# 🔵 GP2: General Purpose SSD

GP2 is the previous generation of General Purpose SSD.

Unlike GP3, GP2 performance is closely connected to the size of the volume.

The basic relationship is:

```text
Volume Size
     │
     ▼
Baseline IOPS
```

For GP2, baseline performance scales at:

```text
3 IOPS
per
GiB
```

up to the maximum GP2 performance.

---

# 📏 GP2 Capacity

GP2 supports:

```text
1 GiB
  │
  ▼
16 TiB
```

and provides up to:

```text
16,000 IOPS
```

per volume.

---

# 🧮 GP2 Performance Example

Suppose I create:

```text
100 GiB GP2
```

The baseline performance is approximately:

```text
100 GiB
   ×
3 IOPS/GiB

=

300 IOPS
```

If I increase the volume:

```text
1,000 GiB
    ×
3 IOPS/GiB

=

3,000 IOPS
```

So with GP2:

> Increasing storage capacity also increases baseline IOPS.

---

# 💳 GP2 Burst Credits

Smaller GP2 volumes can temporarily burst above their baseline performance using an I/O credit system.

Conceptually:

```text
Normal Workload
      │
      ▼
Earn Credits
      │
      ▼
Credit Balance
      │
      ▼
Temporary Heavy Workload
      │
      ▼
Use Credits
      │
      ▼
Burst Performance
```

GP2 volumes below 1 TiB can burst up to:

```text
3,000 IOPS
```

when sufficient I/O credits are available.

---

# 🧠 Why GP2 Credits Matter

Imagine a smaller GP2 volume with a low baseline:

```text
Baseline
   │
   ├──── Normal Activity
   │
   │
   └───────────────┐
                   │
                   ▼
              Heavy Activity
                   │
                   ▼
              Use Credits
                   │
                   ▼
            Up to 3,000 IOPS
```

This works well for workloads that occasionally need more performance.

However, if the workload continuously exceeds baseline performance, credits can be depleted.

Performance then returns toward the baseline level.

This makes GP2 performance less predictable for sustained workloads on smaller volumes.

---

# 🆚 GP2 vs GP3

This comparison made the difference much clearer to me.

| Feature                      |                      GP2 |               GP3 |
| ---------------------------- | -----------------------: | ----------------: |
| Storage                      |                      SSD |               SSD |
| Volume size                  |           1 GiB - 16 TiB |    1 GiB - 64 TiB |
| Baseline IOPS                |     Based on volume size |             3,000 |
| Maximum IOPS                 |                   16,000 |            80,000 |
| Maximum throughput           |                250 MiB/s |       2,000 MiB/s |
| Burst credits                | Yes, for smaller volumes |                No |
| Performance tied to capacity |                      Yes | Largely decoupled |
| Boot volume                  |                      Yes |               Yes |

The key difference is:

```text
GP2

Need More Performance
       │
       ▼
Often Need More Capacity


GP3

Need More Performance
       │
       ▼
Provision Performance
Independently
```

---

# 💰 Why GP3 is Usually Preferred

AWS recommends GP3 for most General Purpose SSD workloads.

GP3 has several advantages:

```text
Predictable Baseline Performance
             +
Independent IOPS Configuration
             +
Independent Throughput Configuration
             +
Lower Storage Price per GiB than GP2
```

AWS states that GP3 storage pricing is approximately **20% lower per GiB than GP2**.

Therefore, for a new general-purpose workload, GP3 is normally the first General Purpose SSD option I would evaluate.

---

# 🔴 Provisioned IOPS SSD

General Purpose SSD works well for many applications.

But some workloads need:

```text
Consistently High IOPS

Very Low Latency

High Throughput

Predictable Storage Performance
```

This is where:

```text
Provisioned IOPS SSD
```

becomes important.

AWS provides:

```text
Provisioned IOPS SSD
│
├── io2 Block Express
│
└── io1
```

These are the highest-performance EBS SSD volume types and are designed for critical IOPS-intensive and throughput-intensive workloads.

---

# 🎯 Provisioned IOPS Use Cases

Provisioned IOPS SSD is designed for workloads where storage performance is a critical requirement.

Examples include:

```text
High-Performance Databases

I/O-Intensive Databases

Mission-Critical Applications

Applications Sensitive to
Storage Latency and Consistency
```

Examples AWS specifically identifies for io2 Block Express include:

```text
Oracle

SAP HANA

Microsoft SQL Server

SAS Analytics
```

---

# 🟣 IO2 Block Express

IO2 Block Express is the highest-performance EBS volume option.

An important current AWS update is:

> As of April 30, 2025, all new and previously created `io2` volumes are `io2` Block Express volumes.

Therefore, it is no longer useful to think of ordinary `io2` and `io2 Block Express` as two separate current options.

The current model is:

```text
Provisioned IOPS SSD
│
├── io2 Block Express
│
└── io1
```

---

# ⚡ IO2 Block Express Performance

IO2 Block Express supports up to:

```text
256,000 IOPS
```

per volume on supported Nitro-based instances.

It also supports throughput up to:

```text
4,000 MiB/s
```

and storage capacity up to:

```text
64 TiB
```

---

# 📊 IO2 Block Express Performance Model

```text
IO2 Block Express

Capacity
4 GiB → 64 TiB

IOPS
100 → 256,000

Throughput
Up to 4,000 MiB/s
```

The maximum IOPS-to-size ratio is:

```text
1,000 IOPS
per
GiB
```

Therefore:

```text
256 GiB
   ×
1,000 IOPS/GiB

=

256,000 IOPS
```

A volume of at least 256 GiB can therefore be provisioned for the maximum 256,000 IOPS, assuming the attached EC2 instance supports that level of EBS performance.

---

# ⏱️ IO2 Block Express Latency

One of the strongest reasons to consider IO2 Block Express is latency.

AWS designs IO2 Block Express to provide:

```text
Average latency
under 500 microseconds
for 16 KiB I/O
```

when attached to supported Nitro-based instances.

This makes it suitable for storage-latency-sensitive workloads where predictable performance is important.

---

# 🛡️ IO2 Durability

IO2 Block Express is also designed for higher durability than General Purpose SSD and IO1.

AWS designs IO2 Block Express for:

```text
99.999%
Volume Durability
```

with an annual failure rate no higher than:

```text
0.001%
```

This makes it suitable for mission-critical workloads requiring both high performance and high durability.

---

# 🔵 IO1

IO1 is an earlier generation Provisioned IOPS SSD volume.

It is designed for I/O-intensive workloads, especially databases that require sustained and predictable IOPS.

IO1 supports:

```text
Volume Size
4 GiB → 16 TiB

Provisioned IOPS
100 → 64,000

Throughput
Up to 1,000 MiB/s
```

---

# 🧮 IO1 IOPS-to-Size Ratio

IO1 supports a maximum ratio of:

```text
50 IOPS
per
GiB
```

For example:

```text
100 GiB
   ×
50 IOPS/GiB

=

5,000 IOPS
```

To provision the maximum:

```text
64,000 IOPS
```

the volume must be at least:

```text
1,280 GiB
```

because:

```text
1,280 GiB × 50

=

64,000 IOPS
```

---

# 🚚 IO1 Throughput

IO1 provides throughput up to:

```text
1,000 MiB/s
```

Maximum throughput requires the appropriate IOPS configuration and a supported Nitro-based EC2 instance.

IO1 can provide up to 64,000 IOPS on Nitro-based instances.

On other supported instances, achievable performance is lower.

---

# 🆚 IO1 vs IO2 Block Express

| Feature            |                      IO1 |                         IO2 Block Express |
| ------------------ | -----------------------: | ----------------------------------------: |
| Storage            |                      SSD |                                       SSD |
| Minimum size       |                    4 GiB |                                     4 GiB |
| Maximum size       |                   16 TiB |                                    64 TiB |
| Maximum IOPS       |                   64,000 |                                   256,000 |
| Maximum IOPS/GiB   |                     50:1 |                                   1,000:1 |
| Maximum throughput |              1,000 MiB/s |                               4,000 MiB/s |
| Multi-Attach       |                      Yes |                                       Yes |
| Boot volume        |                      Yes |                                       Yes |
| Durability         |              99.8%-99.9% |                                   99.999% |
| Primary fit        | Sustained IOPS workloads | Most demanding mission-critical workloads |

AWS recommends IO2 over IO1 when possible because IO2 provides better:

```text
Performance
    +
Consistency
    +
Durability
    +
Cost characteristics
```

---

# 🖥️ Nitro System and EBS Performance

The EC2 instance itself must be capable of delivering the performance configured on the EBS volume.

This is an important lesson:

```text
Fast EBS Volume
       +
Insufficient EC2 EBS Bandwidth
       =
Performance Bottleneck
```

Nitro-based EC2 instances support the highest EBS performance levels.

For IO2 Block Express:

```text
Nitro-based Instance
        │
        ▼
Up to 256,000 IOPS
```

Other supported instances can be attached to volumes provisioned with up to 64,000 IOPS, but AWS documents achievable performance of up to 32,000 IOPS.

Therefore:

> Selecting a high-performance EBS volume alone does not guarantee that the application will achieve that performance.

The EC2 instance's EBS limits also matter.

---

# 🔗 Multi-Attach

Provisioned IOPS SSD volumes also support:

```text
EBS Multi-Attach
```

Multi-Attach allows one supported EBS volume to be attached to multiple Nitro-based EC2 instances in the same Availability Zone.

```text
EC2-A ──┐
        │
EC2-B ──┼──► Provisioned IOPS Volume
        │
EC2-C ──┘
```

AWS currently supports Multi-Attach for:

```text
io1

and

io2
```

volumes.

A Multi-Attach enabled volume can be attached to up to:

```text
16 Nitro-based EC2 instances
```

in the same Availability Zone.

---

# ⚠️ Multi-Attach Does Not Mean Normal Shared File Storage

This is an important distinction.

```text
Multi-Attach
      ≠
Automatically Safe
Shared File System
```

Every attached instance receives read and write access to the shared block device.

Applications must coordinate concurrent writes correctly.

Standard file systems such as:

```text
EXT4

XFS
```

are not designed to be simultaneously accessed by multiple servers in this way.

For production shared-block-storage architectures, an appropriate clustered file system or application-level coordination is required.

---

# 🧠 Choosing Between GP3 and IO2

The most useful decision for me is not:

> Which one is faster?

Instead:

> What storage performance does my application actually require?

For many applications:

```text
Web Server

Application Server

Development Environment

Virtual Desktop

Medium Database
```

start by evaluating:

```text
GP3
```

If the workload requires:

```text
Sustained High IOPS

Very Low Latency

More than 80,000 IOPS

More than 2,000 MiB/s

Higher Volume Durability
```

then evaluate:

```text
IO2 Block Express
```

---

# 📊 SSD EBS Volume Comparison

| Feature                             |                             GP2 |                            GP3 |                      IO1 |                           IO2 Block Express |
| ----------------------------------- | ------------------------------: | -----------------------------: | -----------------------: | ------------------------------------------: |
| Type                                |                 General Purpose |                General Purpose |         Provisioned IOPS |                            Provisioned IOPS |
| Minimum size                        |                           1 GiB |                          1 GiB |                    4 GiB |                                       4 GiB |
| Maximum size                        |                          16 TiB |                         64 TiB |                   16 TiB |                                      64 TiB |
| Maximum IOPS                        |                          16,000 |                         80,000 |                   64,000 |                                     256,000 |
| Maximum throughput                  |                       250 MiB/s |                    2,000 MiB/s |              1,000 MiB/s |                                 4,000 MiB/s |
| Performance independent of capacity |                               ❌ |                              ✅ |              Provisioned |                                 Provisioned |
| Burst credits                       |                             Yes |                             No |                       No |                                          No |
| Multi-Attach                        |                               ❌ |                              ❌ |                        ✅ |                                           ✅ |
| Boot volume                         |                               ✅ |                              ✅ |                        ✅ |                                           ✅ |
| Best fit                            | Older general-purpose workloads | Most general-purpose workloads | Sustained IOPS workloads | Mission-critical high-performance workloads |

---

# 🧠 Simple Selection Model

```text
What does the workload need?
           │
           ▼
General Purpose Storage?
           │
          Yes
           │
           ▼
          GP3


Need sustained high IOPS
or very low latency?
           │
          Yes
           │
           ▼
Provisioned IOPS
           │
           ▼
     Prefer IO2
     Block Express
```

GP2 and IO1 remain available, but for new architectures I would first evaluate their newer alternatives:

```text
GP2 → GP3

IO1 → IO2 Block Express
```

---

# 🏗️ Example 1: Standard Web Application

```text
Internet
    │
    ▼
Load Balancer
    │
    ▼
EC2 Web Server
    │
    ▼
GP3
```

The application needs reliable general-purpose storage but does not have extreme storage latency requirements.

GP3 is a good starting point.

---

# 🏗️ Example 2: Medium Single-Instance Database

```text
Application
     │
     ▼
EC2
     │
     ▼
GP3
```

If the database requirements fit within GP3 performance limits, there may be no reason to immediately choose Provisioned IOPS.

This reinforces an important cost principle:

> Do not provision high-performance storage simply because the workload is called a database.

Measure what the database actually needs.

---

# 🏗️ Example 3: Mission-Critical Database

```text
Application
     │
     ▼
EC2 Nitro Instance
     │
     ▼
IO2 Block Express
     │
     ├── High IOPS
     ├── High Throughput
     └── Low Latency
```

A performance-sensitive database requiring sustained storage performance may justify IO2 Block Express.

---

# ⚠️ Common Mistakes

### Mistake 1: Choosing Storage Only by Capacity

```text
"I need 500 GiB"
```

is not enough information.

Also ask:

```text
How many IOPS?

How much throughput?

What latency?

How consistent must performance be?

How critical is the data?
```

---

### Mistake 2: Assuming GP2 and GP3 Work the Same Way

They do not.

```text
GP2
 │
 └── Performance linked to capacity
     + burst credits


GP3
 │
 └── Performance configured
     independently
```

---

### Mistake 3: Increasing GP3 Capacity Just to Increase Performance

GP3 allows IOPS and throughput to be configured independently within supported ratios.

Do not provision unnecessary storage capacity solely to obtain performance unless the ratio requirements require it.

---

### Mistake 4: Assuming Every Database Needs Provisioned IOPS

A small or medium database may work perfectly well with GP3.

Start with workload requirements.

---

### Mistake 5: Provisioning 256,000 IOPS Without Checking EC2

The EC2 instance must support the required EBS performance.

```text
EBS Performance
      │
      ▼
EC2 EBS Capability
      │
      ▼
Actual Application Performance
```

---

### Mistake 6: Treating Multi-Attach as Shared File Storage

Multi-Attach provides multiple instances access to the same block device.

The application or clustered file system must manage concurrent access safely.

---

# ✅ Best Practices

* Use GP3 as the starting point for most new general-purpose EBS workloads.
* Understand IOPS, throughput, and latency separately.
* Measure application storage requirements before choosing a volume.
* Provision only the performance the workload requires.
* Consider IO2 Block Express for mission-critical, I/O-intensive, latency-sensitive workloads.
* Match high-performance EBS volumes with EC2 instances capable of delivering the required EBS bandwidth and IOPS.
* Prefer Nitro-based instances when very high EBS performance is required.
* Monitor EBS performance using Amazon CloudWatch.
* Review `VolumeReadOps`, `VolumeWriteOps`, throughput, latency-related behavior, and workload characteristics before resizing storage.
* Use Multi-Attach only when the application and file system are designed for shared block storage.
* Evaluate both performance and cost before moving from General Purpose SSD to Provisioned IOPS SSD.

---

# ❓ Interview Questions

### Q1. What are the General Purpose SSD EBS volume types?

**Answer**

```text
GP3

and

GP2
```

GP3 is the newer General Purpose SSD option and allows performance to be provisioned independently from capacity.

---

### Q2. What baseline performance does GP3 provide?

**Answer**

GP3 currently includes:

```text
3,000 IOPS

and

125 MiB/s throughput
```

as baseline performance.

---

### Q3. Does GP3 use burst credits?

**Answer**

No.

GP3 can indefinitely sustain its provisioned IOPS and throughput performance.

---

### Q4. What is the maximum current GP3 performance?

**Answer**

AWS currently documents up to:

```text
80,000 IOPS

and

2,000 MiB/s
```

per GP3 volume, subject to volume-size ratios and EC2 instance capabilities.

---

### Q5. How does GP2 determine baseline IOPS?

**Answer**

GP2 baseline performance scales with volume capacity at approximately:

```text
3 IOPS per GiB
```

up to its maximum of 16,000 IOPS.

---

### Q6. What is the major difference between GP2 and GP3?

**Answer**

GP2 performance is closely linked to volume capacity and smaller volumes use burst credits.

GP3 separates capacity from provisioned IOPS and throughput and does not use burst credits.

---

### Q7. When should Provisioned IOPS SSD be considered?

**Answer**

For critical workloads requiring sustained high IOPS, high throughput, predictable storage performance, or very low latency.

---

### Q8. What Provisioned IOPS volume types does EBS provide?

**Answer**

```text
IO2 Block Express

and

IO1
```

---

### Q9. Is IO2 different from IO2 Block Express today?

**Answer**

For current EBS usage, all IO2 volumes are IO2 Block Express.

AWS states that as of April 30, 2025, all new and previously created IO2 volumes are IO2 Block Express volumes.

---

### Q10. What is the maximum IO2 Block Express performance?

**Answer**

On supported Nitro-based EC2 instances, IO2 Block Express supports up to:

```text
256,000 IOPS

and

4,000 MiB/s throughput
```

per volume.

---

### Q11. What is the maximum IO2 volume size?

**Answer**

```text
64 TiB
```

---

### Q12. What latency is IO2 Block Express designed to provide?

**Answer**

AWS designs IO2 Block Express to provide average latency below:

```text
500 microseconds
```

for 16 KiB I/O operations when used with supported Nitro-based instances.

---

### Q13. What is the maximum IO1 performance?

**Answer**

IO1 supports up to:

```text
64,000 IOPS

and

1,000 MiB/s
```

on supported configurations.

---

### Q14. Why does the EC2 instance type matter for EBS performance?

**Answer**

The EC2 instance has its own EBS bandwidth, throughput, and IOPS limits.

Provisioning a high-performance EBS volume does not guarantee that the EC2 instance can consume all of that performance.

---

### Q15. Which EBS volume types support Multi-Attach?

**Answer**

Multi-Attach is supported for:

```text
IO1

and

IO2
```

Provisioned IOPS SSD volumes.

---

### Q16. How many EC2 instances can use a Multi-Attach enabled volume?

**Answer**

Up to:

```text
16 Nitro-based instances
```

in the same Availability Zone, subject to AWS's supported configurations.

---

### Q17. Does Multi-Attach automatically make an EBS volume a shared file system?

**Answer**

No.

It provides shared block-device access. The application or an appropriate clustered file system must coordinate concurrent access and maintain data consistency.

---

### Q18. For a new general-purpose workload, would you start with GP2 or GP3?

**Answer**

I would normally evaluate GP3 first because it provides predictable baseline performance, allows independent configuration of capacity and performance, and AWS prices GP3 storage lower per GiB than GP2.

The final decision should still be based on the workload requirements.

---

# 💡 Key Takeaways

* EBS volume selection should be based on workload requirements, not only storage capacity.
* IOPS measures the number of storage operations.
* Throughput measures the amount of data transferred.
* Latency measures how long an I/O operation takes.
* GP3 is the current General Purpose SSD option I would evaluate first for most workloads.
* GP3 provides 3,000 baseline IOPS and 125 MiB/s baseline throughput.
* GP3 currently scales to 80,000 IOPS and 2,000 MiB/s.
* GP3 does not use burst credits.
* GP2 performance scales with volume capacity and smaller volumes use burst credits.
* Provisioned IOPS SSD is designed for storage-performance-sensitive workloads.
* IO2 Block Express is the current high-performance EBS option for demanding mission-critical workloads.
* IO2 Block Express supports up to 256,000 IOPS and 4,000 MiB/s on supported Nitro configurations.
* IO2 Block Express provides significantly higher durability than GP3, GP2, and IO1.
* IO1 remains available but IO2 is generally the Provisioned IOPS option to evaluate first for new workloads.
* High EBS performance also requires an EC2 instance capable of consuming that performance.
* IO1 and IO2 support EBS Multi-Attach.
* Multi-Attach provides shared block access, not automatically safe shared file storage.
* The right question is not **"Which EBS volume is fastest?"** but **"What storage performance does my workload actually require?"**

---

# 📚 Related Topics

* Amazon EBS
* EBS Snapshots
* EC2 Instance Store
* EBS Encryption
* Amazon EBS Multi-Attach
* EBS-Optimized EC2 Instances
* AWS Nitro System
* Amazon CloudWatch
* Amazon EFS
* Database Storage Design
* AWS Cost Optimization

---

# 📖 References

* AWS Documentation: Amazon EBS Volume Types
* AWS Documentation: General Purpose SSD Volumes
* AWS Documentation: Provisioned IOPS SSD Volumes
* AWS Documentation: IO2 Block Express
* AWS Documentation: Amazon EBS Multi-Attach
* AWS Documentation: Amazon EBS I/O Characteristics and Monitoring
* AWS Documentation: Amazon EBS Optimization
