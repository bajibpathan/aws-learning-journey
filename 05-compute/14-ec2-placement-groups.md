# 🖥️ Amazon EC2 Placement Groups

> EC2 Placement Groups allow me to influence how groups of EC2 instances are physically placed relative to each other to meet requirements such as low latency, high network throughput, hardware-failure isolation, or precise time synchronization.

---

# 📖 Overview

Normally, when I launch an EC2 instance, I choose things such as:

```text
AWS Region
    │
    ▼
Availability Zone
    │
    ▼
Subnet
    │
    ▼
EC2 Instance
```

AWS determines the underlying physical infrastructure where the instance runs.

For most applications, this is exactly what I want.

But some applications have special requirements.

For example:

```text
HPC Cluster
     │
     ▼
Instances should be close together
for low latency


Critical Servers
     │
     ▼
Instances should be separated
to reduce correlated failures


Distributed Database
     │
     ▼
Groups of instances should be
isolated across hardware
```

This is where:

```text
EC2 Placement Groups
```

become useful.

---

# 🎯 What Is a Placement Group?

A placement group is a logical grouping of EC2 instances that lets me influence how AWS places those instances on the underlying infrastructure.

Without a placement group:

```text
EC2-A ──► AWS chooses placement

EC2-B ──► AWS chooses placement

EC2-C ──► AWS chooses placement
```

With a placement group:

```text
EC2 Instances
      │
      ▼
Placement Strategy
      │
      ▼
AWS places instances
according to that strategy
```

Placement groups are optional.

If I do not use one, EC2 normally places instances across underlying hardware in a way designed to reduce correlated failures.

---

# 🏗️ Placement Strategies

The traditional strategies I need to understand are:

```text
EC2 Placement Groups
│
├── Cluster
│
├── Spread
│
└── Partition
```

Current AWS also provides:

```text
Precision Time
```

So the current set is:

```text
Placement Groups
│
├── Cluster
├── Spread
├── Partition
└── Precision Time
```

The first three answer questions about **where instances should be positioned relative to each other**.

Precision Time addresses a different requirement: placing supported instances on infrastructure with access to higher-precision AWS time sources.

---

# 1. 🚀 Cluster Placement Group

The easiest way for me to remember Cluster is:

> **Keep the instances close together.**

A Cluster Placement Group packs instances close together within:

```text
One Availability Zone
```

Conceptually:

```text
AWS Region
│
├── AZ-A
│    │
│    └── Cluster Placement Group
│          │
│          ├── EC2-A
│          ├── EC2-B
│          ├── EC2-C
│          └── EC2-D
│
├── AZ-B
│
└── AZ-C
```

A Cluster Placement Group **cannot span multiple Availability Zones**.

---

# 🎯 Why Cluster Instances Together?

Some applications have large amounts of communication between EC2 instances.

For example:

```text
EC2-A ◄────► EC2-B
  ▲            ▲
  │            │
  ▼            ▼
EC2-C ◄────► EC2-D
```

If the application requires:

```text
Low Network Latency

High Network Throughput

High Packet-Per-Second Performance
```

then keeping instances close together can improve network performance.

AWS places instances in a Cluster Placement Group within the same high-bisection-bandwidth network segment.

---

# ⚡ Cluster Placement Group Use Cases

Cluster Placement Groups are useful for tightly coupled applications such as:

```text
High-Performance Computing

Scientific Simulations

Parallel Processing

High-Performance Analytics

Tightly Coupled Compute Clusters
```

The common characteristic is:

```text
A lot of communication
between EC2 instances
```

---

# 🌐 Cluster Networking

The architecture is approximately:

```text
        Cluster Placement Group

      ┌─────────────────────┐
      │                     │
      │ EC2-A ◄────► EC2-B  │
      │   ▲            ▲    │
      │   │            │    │
      │   ▼            ▼    │
      │ EC2-C ◄────► EC2-D  │
      │                     │
      └─────────────────────┘

          Single AZ
```

AWS currently documents a higher single-flow TCP/IP throughput limit between supported enhanced-networking instances inside a Cluster Placement Group.

For high-performance workloads, instance networking capability still matters.

A placement group cannot make a low-network-performance instance type behave like a high-network-performance instance.

---

# ✅ Cluster Placement Recommendations

AWS recommends:

```text
Launch Required Instances Together
              +
Prefer Same Instance Type
              +
Use Enhanced Networking
```

Why launch the instances together?

Imagine I need:

```text
20 EC2 Instances
```

but initially launch only:

```text
5 Instances
```

AWS finds suitable nearby capacity for those five.

Later I try adding another 15.

There may no longer be sufficient capacity available in that placement group.

Therefore:

```text
Known Cluster Size
      │
      ▼
Launch Together
      │
      ▼
Better Chance of Capacity
```

---

# ⚠️ Cluster Capacity Errors

Because Cluster Placement Groups have stricter placement requirements, I may encounter:

```text
Insufficient Capacity
```

For example:

```text
Cluster Placement Group
        │
        ├── EC2
        ├── EC2
        ├── EC2
        └── Need More EC2
                │
                ▼
         No Suitable Capacity
```

AWS also supports using an On-Demand Capacity Reservation with Cluster Placement Groups when explicit capacity reservation is required.

---

# 🧠 Cluster Mental Model

```text
CLUSTER
   =
CLOSE
```

Think:

```text
Close Together
      │
      ▼
Low Latency
      +
High Throughput
```

---

# 2. 🛡️ Spread Placement Group

Spread takes almost the opposite approach.

Instead of:

```text
Keep instances together
```

the goal is:

```text
Keep instances apart
```

A Spread Placement Group places instances on distinct underlying hardware.

Conceptually:

```text
          Spread Placement Group

Rack A          Rack B          Rack C
  │               │               │
  ▼               ▼               ▼
EC2-A           EC2-B           EC2-C
```

Each rack has independent infrastructure such as:

```text
Power

Networking
```

This reduces the possibility that one underlying hardware failure affects multiple critical instances in the group.

---

# 🌎 Spread Across Availability Zones

Unlike Cluster Placement Groups, rack-level Spread Placement Groups can span:

```text
Multiple Availability Zones
```

For example:

```text
AWS Region
│
├── AZ-A
│    ├── Rack 1 → EC2-A
│    └── Rack 2 → EC2-B
│
├── AZ-B
│    ├── Rack 3 → EC2-C
│    └── Rack 4 → EC2-D
│
└── AZ-C
     ├── Rack 5 → EC2-E
     └── Rack 6 → EC2-F
```

This provides both:

```text
Hardware Separation
        +
AZ Distribution
```

when designed appropriately.

---

# 🎯 Spread Placement Use Case

Spread is intended for a:

```text
Small Number
of
Critical Instances
```

that should not share underlying hardware.

Example:

```text
Critical Server A
        │
        ▼
      Rack A


Critical Server B
        │
        ▼
      Rack B


Critical Server C
        │
        ▼
      Rack C
```

If Rack A fails:

```text
Rack A ❌

Rack B ✅

Rack C ✅
```

the other critical instances remain on separate hardware.

---

# 🏢 Example: Domain Controllers

Suppose I have several critical identity servers.

Without deliberate separation:

```text
Rack A
│
├── DC-1
├── DC-2
└── DC-3
```

Rack failure:

```text
Rack A ❌
    │
    ▼
All Domain Controllers ❌
```

With Spread:

```text
Rack A        Rack B        Rack C
  │             │             │
 DC-1          DC-2          DC-3
```

Now:

```text
Rack A ❌
  │
 DC-1 ❌

DC-2 ✅
DC-3 ✅
```

This is the type of failure-isolation requirement Spread Placement Groups are designed to address.

---

# 🔢 Important Spread Limit

For rack-level Spread Placement Groups in AWS Regions:

```text
Maximum
7 Running Instances
per Availability Zone
per Spread Placement Group
```

For example:

```text
AZ-A
7 instances

AZ-B
7 instances

AZ-C
7 instances
```

could allow:

```text
21 instances
```

in that Spread Placement Group across those three AZs.

---

# 🧠 Spread Mental Model

```text
SPREAD
   =
SEPARATE
```

Think:

```text
Small Number
      +
Critical Instances
      +
Separate Hardware
```

---

# 🏢 Spread and AWS Outposts

AWS also supports Spread Placement Groups on:

```text
AWS Outposts
```

Rack-level spread is supported in AWS Regions and Outposts.

Host-level spread is specifically available on AWS Outposts.

Conceptually:

```text
AWS Outpost
│
├── Host A → EC2
├── Host B → EC2
└── Host C → EC2
```

This allows workloads running on AWS infrastructure at an organization's site to be deliberately distributed across hosts.

---

# 3. 🧩 Partition Placement Group

Partition Placement Groups combine:

```text
Separation
      +
Scale
```

Instead of putting every instance on completely separate hardware, EC2 divides the placement group into:

```text
Partitions
```

Each partition receives its own set of racks.

Instances in different partitions do not share the same racks.

---

# 🏗️ Partition Architecture

Imagine:

```text
Partition Placement Group
│
├── Partition 1
│    ├── EC2-A
│    ├── EC2-B
│    └── EC2-C
│
├── Partition 2
│    ├── EC2-D
│    ├── EC2-E
│    └── EC2-F
│
└── Partition 3
     ├── EC2-G
     ├── EC2-H
     └── EC2-I
```

The important rule is:

```text
Partition 1 racks
      ≠
Partition 2 racks
      ≠
Partition 3 racks
```

Instances inside the same partition can share racks.

But different partitions do not share racks.

---

# 💥 Hardware Failure Isolation

Suppose:

```text
Partition 1
     │
     ▼
Rack Failure
```

The goal is to contain the hardware failure within:

```text
Partition 1
```

rather than affecting instances in:

```text
Partition 2

Partition 3
```

This allows distributed applications to design replication around the infrastructure topology.

---

# 🌎 Partition Groups Across AZs

Partition Placement Groups can span multiple Availability Zones in the same Region.

For example:

```text
AWS Region
│
├── AZ-A
│    ├── Partition 1
│    ├── Partition 2
│    └── Partition 3
│
└── AZ-B
     ├── Partition 1
     ├── Partition 2
     └── Partition 3
```

The exact design depends on the workload.

---

# 🔢 Partition Limit

A Partition Placement Group supports:

```text
Maximum
7 Partitions
per Availability Zone
```

This is different from Spread.

Remember:

```text
Spread
   │
   ▼
7 INSTANCES per AZ


Partition
   │
   ▼
7 PARTITIONS per AZ
```

Each partition can contain multiple instances.

The number of instances is ultimately subject to the account's EC2 limits.

---

# 🎯 Partition Placement Use Cases

Partition Placement Groups are useful for large distributed systems such as:

```text
HDFS

HBase

Cassandra

Kafka
```

These applications often replicate data across nodes.

If the application understands which partition each node belongs to, it can make better replication decisions.

For example:

```text
Data Copy 1
     │
     ▼
Partition 1


Data Copy 2
     │
     ▼
Partition 2


Data Copy 3
     │
     ▼
Partition 3
```

Now a rack failure affecting one partition is less likely to remove every copy of the data.

---

# 🧠 Topology-Aware Applications

One important capability of Partition Placement Groups is:

```text
Partition Visibility
```

Applications can determine which partition an instance belongs to.

This is useful for:

```text
Topology-Aware
Distributed Applications
```

The application can make decisions such as:

```text
Do not place all replicas
in the same partition.
```

---

# 🧠 Partition Mental Model

```text
PARTITION
    =
GROUPS OF SEPARATED INSTANCES
```

Think:

```text
Large Distributed Application
            +
Hardware Failure Isolation
            +
Topology Awareness
```

---

# 4. ⏱️ Precision Time Placement Group

This is a newer placement strategy that was not covered in the original lesson.

AWS now also supports:

```text
Precision Time Placement Groups
```

These place supported EC2 instances on infrastructure with direct access to higher-precision AWS time sources.

This is useful when accurate synchronization between systems is especially important.

Examples include:

```text
Distributed Databases

Financial Systems

Transaction Ordering

Distributed Event Processing

Precise Timestamping
```

Conceptually:

```text
EC2-A ──┐
EC2-B ──┼──► High-Precision Time Source
EC2-C ──┘
```

Supported instances can use an enhanced Amazon Time Sync Service, and supported Linux instances can also access precision-time capabilities such as a PTP Hardware Clock.

This strategy solves a different problem from Cluster, Spread, and Partition:

```text
Cluster
   │
   ▼
Network Performance


Spread
   │
   ▼
Instance Isolation


Partition
   │
   ▼
Group Isolation


Precision Time
   │
   ▼
Clock Synchronization
```

---

# 🆚 Placement Group Comparison

| Strategy       | Primary Goal                         | AZ Scope                       | Typical Workload                 |
| -------------- | ------------------------------------ | ------------------------------ | -------------------------------- |
| Cluster        | Low latency and high throughput      | Single AZ                      | HPC, tightly coupled compute     |
| Spread         | Separate critical instances          | Can span AZs                   | Small number of critical servers |
| Partition      | Separate groups of instances         | Can span AZs                   | HDFS, Cassandra, Kafka           |
| Precision Time | High-precision clock synchronization | Depends on supported placement | Financial/distributed systems    |

---

# 🎯 How Do I Choose?

Ask:

```text
What is my main requirement?
```

### Need maximum network performance between instances?

```text
Low Latency
    +
High Throughput
    │
    ▼
CLUSTER
```

### Need a few critical instances physically separated?

```text
Critical Instances
      +
Hardware Separation
      │
      ▼
SPREAD
```

### Need many distributed instances separated into failure domains?

```text
Large Distributed System
          +
Partitions
          │
          ▼
PARTITION
```

### Need very precise time synchronization?

```text
Microsecond-Level
Time Requirements
       │
       ▼
PRECISION TIME
```

---

# 🏗️ Architecture Comparison

```text
CLUSTER

┌─────────────────────┐
│ EC2 EC2 EC2 EC2     │
│                     │
│ Close Together      │
└─────────────────────┘

Goal:
Performance
```

```text
SPREAD

Rack A     Rack B     Rack C
  │          │          │
 EC2        EC2        EC2

Goal:
Separate critical instances
```

```text
PARTITION

Partition A     Partition B     Partition C
│               │               │
├─ EC2           ├─ EC2          ├─ EC2
├─ EC2           ├─ EC2          ├─ EC2
└─ EC2           └─ EC2          └─ EC2

Goal:
Separate groups
```

---

# ⚠️ Placement Groups Do Not Replace Multi-AZ Architecture

A particularly important architecture lesson is:

```text
Cluster Placement Group
        │
        ▼
Single AZ
```

Therefore, Cluster improves:

```text
Network Performance
```

but it does not provide:

```text
Multi-AZ Resilience
```

These are different architectural goals.

For example:

```text
Performance Requirement
       │
       ▼
Cluster Placement


Availability Requirement
       │
       ▼
Multi-AZ Architecture
```

Sometimes architecture requires balancing both.

---

# ⚠️ Placement Groups Do Not Guarantee Unlimited Performance

A Cluster Placement Group can improve the networking characteristics between instances.

But performance still depends on:

```text
Instance Type

Network Bandwidth

Enhanced Networking

Application Architecture

Traffic Pattern

Operating System

Protocol
```

So:

```text
Cluster Placement Group
          ≠
Unlimited Network Performance
```

---

# 💰 Placement Group Pricing

Creating a placement group itself does not incur an additional placement-group charge.

However, I still pay for the underlying resources such as:

```text
EC2 Instances

EBS

Network Transfer

Other AWS Services
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Cluster Can Span Multiple AZs

It cannot.

```text
Cluster
   =
Single AZ
```

---

## Mistake 2: Thinking Spread Means Different AZs Only

Spread is specifically about:

```text
Distinct Underlying Hardware
```

A rack-level Spread Placement Group can also span multiple AZs.

---

## Mistake 3: Confusing Spread and Partition Limits

Remember:

```text
Spread
   =
7 running instances
per AZ per group


Partition
   =
7 partitions
per AZ
```

---

## Mistake 4: Thinking Every Instance in a Partition Is on Separate Hardware

Not necessarily.

Instances inside the same partition can share racks.

The isolation exists:

```text
BETWEEN partitions
```

---

## Mistake 5: Using Cluster for High Availability

Cluster is primarily a performance strategy.

It places instances within a single AZ, so it should not be mistaken for a Multi-AZ resilience strategy.

---

## Mistake 6: Thinking Placement Groups Are Mandatory

Most EC2 applications do not require placement groups.

Use them when the workload has a specific placement requirement.

---

## Mistake 7: Thinking There Are Still Only Three Strategies

Older learning material commonly describes:

```text
Cluster

Spread

Partition
```

Current AWS also provides:

```text
Precision Time
```

---

# ✅ Best Practices

* Use placement groups only when the workload has a specific placement requirement.
* Use Cluster for tightly coupled workloads requiring low latency and high network throughput.
* Prefer enhanced-networking-capable instances for Cluster workloads.
* Launch known Cluster capacity together when practical.
* Consider Capacity Reservations when predictable Cluster capacity is important.
* Use Spread for a small number of critical instances requiring hardware separation.
* Remember the seven-running-instances-per-AZ limit for rack-level Spread groups.
* Use Partition for large distributed and replicated systems.
* Design application replication across partitions rather than concentrating replicas in one partition.
* Remember the seven-partitions-per-AZ limit.
* Use Precision Time only when the application has strict clock-synchronization requirements.
* Do not treat Cluster placement as a substitute for Multi-AZ resilience.
* Test workload performance rather than assuming a placement group alone will solve performance problems.

---

# ❓ Interview Questions

### Q1. What is an EC2 Placement Group?

A placement group lets me influence how related EC2 instances are placed relative to each other on AWS infrastructure.

### Q2. What are the current placement strategies?

```text
Cluster

Spread

Partition

Precision Time
```

### Q3. What is a Cluster Placement Group?

It places instances close together within a single Availability Zone to support low-latency and high-throughput communication.

### Q4. Can a Cluster Placement Group span multiple Availability Zones?

No.

### Q5. When would I use Cluster?

For tightly coupled workloads such as HPC where instances communicate heavily with each other.

### Q6. Why should Cluster instances often be launched together?

Because adding instances later can encounter insufficient capacity for the required placement.

### Q7. What is a Spread Placement Group?

It places a small number of critical instances on distinct underlying hardware to reduce correlated hardware failures.

### Q8. Can a rack-level Spread Placement Group span multiple AZs?

Yes.

### Q9. What is the Spread limit?

A rack-level Spread Placement Group supports a maximum of seven running instances per Availability Zone.

### Q10. What is a Partition Placement Group?

It divides instances into logical partitions where different partitions do not share the same racks.

### Q11. Can instances inside the same partition share racks?

Yes.

The hardware separation guarantee is between different partitions.

### Q12. How many partitions can exist per AZ?

Up to:

```text
7 partitions
```

per Availability Zone.

### Q13. Can Partition Placement Groups span multiple AZs?

Yes.

### Q14. What applications commonly benefit from Partition Placement Groups?

Examples include:

```text
HDFS

HBase

Cassandra

Kafka
```

### Q15. Why are Partition Placement Groups useful for distributed databases?

They expose partition topology so applications can distribute replicas across separate failure domains.

### Q16. What is a Precision Time Placement Group?

It places supported instances on infrastructure that provides access to higher-precision AWS time sources.

### Q17. Which strategy should I use for low latency?

```text
Cluster
```

### Q18. Which strategy should I use to separate a few critical instances?

```text
Spread
```

### Q19. Which strategy should I use for a large topology-aware distributed application?

```text
Partition
```

### Q20. Which strategy should I consider when accurate clock synchronization is critical?

```text
Precision Time
```

---

# 💡 Key Takeaways

* Placement Groups influence the physical placement of EC2 instances.
* Placement Groups are optional.
* Cluster keeps instances close together for network performance.
* Cluster Placement Groups are limited to one Availability Zone.
* Cluster is useful for HPC and tightly coupled workloads.
* AWS recommends launching known Cluster capacity together when practical.
* Spread separates critical instances across distinct underlying hardware.
* Rack-level Spread can span multiple Availability Zones.
* Rack-level Spread supports up to seven running instances per AZ per placement group.
* Partition separates groups of instances into independent rack sets.
* Partition Placement Groups can span multiple Availability Zones.
* Partition supports up to seven partitions per AZ.
* Each Partition can contain multiple instances.
* HDFS, HBase, Cassandra, and Kafka are common Partition use cases.
* Current AWS also provides Precision Time Placement Groups.
* Precision Time is designed for workloads requiring highly accurate clock synchronization.
* Placement strategy should be selected according to the application's actual requirement.

The easiest mental model is:

```text
CLUSTER
   =
CLOSE
   =
PERFORMANCE


SPREAD
   =
SEPARATE
   =
CRITICAL INSTANCE ISOLATION


PARTITION
   =
SEPARATE GROUPS
   =
DISTRIBUTED SYSTEMS


PRECISION TIME
   =
SYNCHRONIZED
   =
ACCURATE TIME
```

---

# 📚 Related Topics

* Amazon EC2
* EC2 Instance Types
* Enhanced Networking
* Elastic Network Adapter
* Elastic Fabric Adapter
* High-Performance Computing
* Availability Zones
* Multi-AZ Architecture
* Fault Tolerance
* High Availability
* Capacity Reservations
* AWS Outposts
* Distributed Databases
* Apache Cassandra
* Apache Kafka
* Hadoop HDFS
* Amazon Time Sync Service
