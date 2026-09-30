# 🌍 Amazon Aurora Serverless and Global Databases

> Amazon Aurora provides additional deployment options for workloads that need automatic capacity scaling or database access across multiple AWS Regions. Aurora Serverless focuses on automatically adjusting database capacity, while Aurora Global Database extends Aurora across Regions for low-latency reads, resilience, and disaster recovery.

---

# 📖 Overview

Previously, we looked at:

```text id="b7ufkm"
Provisioned Aurora Clusters
```

With a provisioned cluster, I choose the database instance capacity.

For example:

```text id="i5jm19"
Aurora Cluster
     │
     ├── Instance Class
     ├── CPU
     ├── Memory
     └── I/O Capacity
```

If the workload changes significantly, the database capacity may also need to change.

With:

```text id="12qjcd"
Aurora Serverless
```

AWS can automatically adjust database capacity based on application demand.

The lesson also introduces:

```text id="3jq81a"
Aurora Global Database
```

which extends Aurora across multiple AWS Regions.

These solve different problems:

```text id="y12zlg"
Aurora Serverless
        │
        ▼
Automatic Capacity Scaling


Aurora Global Database
        │
        ▼
Multi-Region Database Architecture
```

---

# PART 1: AMAZON AURORA SERVERLESS

# 🚀 What Is Aurora Serverless?

Aurora Serverless provides an Aurora deployment option where database capacity can automatically adjust according to workload demand.

With a traditional provisioned Aurora cluster:

```text id="30q52n"
Choose Instance Capacity
        │
        ▼
Provision Cluster
        │
        ▼
Application Uses Database
```

If the workload changes:

```text id="fywacv"
Workload Changes
      │
      ▼
Modify Instance Capacity
```

With Aurora Serverless:

```text id="uf4x6f"
Application Demand
       │
       ▼
Aurora Serverless
       │
       ▼
Automatically Adjust
Database Capacity
```

AWS handles the capacity adjustment.

---

# 🎯 Aurora Serverless Use Cases

Aurora Serverless is particularly useful for workloads where database demand changes over time.

The lesson identifies several examples.

## 1. Variable Workloads

Consider a blog that receives traffic only during certain periods.

```text id="p1wnft"
Traffic
  ▲
  │       ███
  │       ███
  │   ██  ███
  │   ██  ███     ██
  │___██__███_____██________► Time
```

Database demand rises and falls throughout the day.

---

## 2. E-Commerce Workloads

An e-commerce application may experience relatively low activity most of the time but receive significantly more traffic during:

```text id="3dlac6"
Sales

Promotions

Special Events

Busy Shopping Periods
```

Conceptually:

```text id="8c4x0c"
Normal Traffic
     │
     ▼
Low DB Demand

        ↓

Promotion Starts
     │
     ▼
Large Traffic Increase
     │
     ▼
High DB Demand

        ↓

Promotion Ends
     │
     ▼
Demand Drops
```

Aurora Serverless can adjust capacity as demand changes.

---

## 3. Unpredictable Workloads

Some applications do not have predictable usage patterns.

```text id="rdnh1w"
Low
 │
 ▼
High
 │
 ▼
Low
 │
 ▼
Very High
 │
 ▼
Medium
```

Rather than provisioning database capacity for the maximum workload all the time, Aurora Serverless can adjust according to demand.

---

## 4. Multi-Tenant Applications

Another use case mentioned in the lesson is:

```text id="bexavh"
Multi-Tenant Applications
```

where database usage can vary depending on activity from different tenants.

---

## 5. Development and Testing

Aurora Serverless can also be useful for:

```text id="30pq7v"
Development

Testing
```

environments where database usage may be intermittent.

---

# 🧮 Aurora Capacity Units

Aurora Serverless capacity is measured using:

```text id="p90myc"
Aurora Capacity Units
        │
        ▼
       ACUs
```

The lesson describes an ACU as representing approximately:

```text id="1xxw71"
~2 GB Memory

+

Corresponding CPU

+

Networking Capacity
```

Instead of selecting a traditional fixed database instance capacity, Aurora Serverless uses ACUs to represent the available database capacity.

---

# 📏 Minimum and Maximum Capacity

I define a capacity range for the serverless database.

For example:

```text id="g2vtue"
Minimum ACU
     │
     │
     │   Scaling Range
     │
     ▼
Maximum ACU
```

Aurora Serverless can then adjust capacity within this range according to application demand.

Conceptually:

```text id="o52b0d"
Low Demand
    │
    ▼
Minimum Capacity

      ↓

Demand Increases
    │
    ▼
Scale Up

      ↓

High Demand
    │
    ▼
Higher Capacity

      ↓

Demand Decreases
    │
    ▼
Scale Down
```

---

# ⚙️ Automatic Capacity Adjustment

The basic idea is:

```text id="tgn0z5"
Application Demand
        │
        ▼
Aurora Serverless
        │
        ▼
Determine Required Capacity
        │
        ▼
Adjust ACUs
```

This removes the need to manually resize the database whenever the workload changes.

---

# 💰 Pay for Consumed Capacity

Aurora Serverless charges based on the database capacity being consumed.

The lesson describes this as:

```text id="ojw3rm"
Pay for Database Capacity
Actually Consumed
```

on a:

```text id="7gvws1"
Per-Second Basis
```

This can make Serverless useful for workloads where demand varies significantly.

---

# ⏸️ Pausing Capacity

The lesson also introduces the ability for a serverless cluster to have:

```text id="0z8hwf"
Zero ACU Capacity
```

allowing capacity to be paused when it is not required.

Conceptually:

```text id="eg61fh"
Application Active
      │
      ▼
Database Capacity
      │
      ▼
Application Inactive
      │
      ▼
0 ACU
```

---

# 📊 Monitoring Aurora Serverless

Capacity should still be monitored.

The lesson mentions metrics such as:

```text id="q73sf8"
Serverless Database Capacity

ACU Utilization
```

These metrics can help understand how much capacity the application is consuming.

---

# 🏊 Warm Pool of Capacity

The lesson describes Aurora Serverless as drawing capacity from a:

```text id="uox9jt"
Warm Pool
```

of available resources.

Conceptually:

```text id="tgj70c"
              AWS Warm Pool
          ┌────┬────┬────┬────┐
          │ACU │ACU │ACU │ACU │
          └────┴────┴────┴────┘
                    │
                    ▼
             Aurora Serverless
                    │
                    ▼
              Application
```

As the application needs more capacity:

```text id="h72vp5"
Demand Increases
      │
      ▼
Additional Capacity
      │
      ▼
Aurora Cluster
```

---

# 🔄 Aurora Serverless Versions

The lesson discusses two versions:

```text id="m8s1mw"
Aurora Serverless v1

Aurora Serverless v2
```

It states that Aurora Serverless v1 reached end of life on:

```text id="zmh7o4"
March 31, 2025
```

and focuses on Serverless v2 going forward.

---

# 🚀 Aurora Serverless v2

Serverless v2 provides capabilities that were not available with the Serverless v1 architecture described in the lesson.

These include:

```text id="r55keb"
Reader DB Instances

Global Databases

IAM Authentication

Performance Insights

Multi-AZ Capabilities
```

---

# 📖 Reader Instances with Serverless v2

With Serverless v2, an Aurora cluster can contain:

```text id="l1t6dg"
Writer Instance

+

Reader Instances
```

Conceptually:

```text id="2bg5iz"
          Aurora Serverless v2
                  │
          ┌───────┴───────┐
          │               │
          ▼               ▼
       Writer           Readers
          │               │
          ▼               ▼
     Read + Write         Read
```

Reader instances can provide horizontal read scaling similar to provisioned Aurora clusters.

---

# 📈 Independent Scaling

With Serverless v1, the lesson describes the cluster as having a single measure of compute capacity that scales between minimum and maximum values.

Conceptually:

```text id="z9xwdf"
Serverless v1

Cluster Capacity
      │
      ▼
Min ───────────── Max
```

With Serverless v2:

```text id="d26fgh"
Serverless v2

Writer
  │
  └──── Min ↔ Max

Reader 1
  │
  └──── Min ↔ Max

Reader 2
  │
  └──── Min ↔ Max
```

The writer and readers can scale within the configured capacity range.

---

# 🧮 Total Serverless v2 Capacity

Because Serverless v2 can contain multiple database instances, total cluster capacity depends on:

```text id="e8by1j"
Configured Capacity Range

+

Number of Writer/Reader Instances

+

Current Capacity of Each Instance
```

Conceptually:

```text id="0zyatq"
Serverless v2 Cluster
│
├── Writer
├── Reader 1
├── Reader 2
└── ...
```

Each can contribute to the overall capacity being consumed.

---

# 🛡️ Serverless v2 and Failover

When the cluster contains reader instances, those instances can also help with failover.

```text id="l2m4mk"
Writer
  │
  │ Failure
  ▼
Reader
  │
  ▼
Failover
```

The lesson highlights this as an important improvement compared with Serverless v1.

---

# 🌎 Serverless v2 and Multi-AZ

Serverless v2 DB instances can be deployed across multiple Availability Zones.

Conceptually:

```text id="h9dpr8"
         Aurora Serverless v2
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
       AZ-A                AZ-B
        │                   │
      Writer              Reader
```

This helps provide business continuity if an Availability Zone experiences a problem.

---

# 🔀 Router / Proxy Fleet

The lesson also describes a:

```text id="uw2mx6"
Router Fleet

or

Proxy Fleet
```

through which applications access the available database resources.

Conceptually:

```text id="h0r89r"
Application
     │
     ▼
Router / Proxy Fleet
     │
     ▼
Aurora Serverless
Database Resources
```

The router fleet scales according to demand.

---

# 🔄 Scaling Through the Router Fleet

As workload increases:

```text id="3en5ko"
Application Demand
       │
       ▼
Router Fleet
       │
       ▼
Additional Capacity
from Warm Pool
       │
       ▼
Aurora Serverless
```

The router fleet can switch active client connections between the available resources as capacity changes.

This helps reduce disruption during scaling operations.

---

# ⚡ Serverless v2 Connection Handling

The lesson explains that Serverless v2 improves this architecture further.

The router fleet can maintain client connections while database capacity:

```text id="ahw7v8"
Scales Up

or

Scales Down
```

Conceptually:

```text id="f4x9dg"
Client Connection
       │
       ▼
Router Fleet
       │
       ▼
Database Capacity
       │
   ┌───┴───┐
   ▼       ▼
Scale Up  Scale Down
```

This allows capacity changes without unnecessarily disrupting active transactions.

---

# 🔬 Granular Scaling

Serverless v2 can scale in increments as small as:

```text id="kjgb1z"
0.5 ACU
```

This provides more granular resource utilization.

For example:

```text id="ky8l06"
2 ACU
  │
  ▼
2.5 ACU
  │
  ▼
3 ACU
  │
  ▼
3.5 ACU
```

rather than requiring large jumps in database capacity.

---

# 🆚 Provisioned Aurora vs Aurora Serverless

| Area                  | Provisioned Aurora                        | Aurora Serverless                 |
| --------------------- | ----------------------------------------- | --------------------------------- |
| Capacity              | Provisioned in advance                    | Automatically adjusted            |
| Instance Sizing       | Select database capacity                  | Uses ACUs                         |
| Variable Workloads    | May require resizing                      | Designed to adjust with demand    |
| Billing               | Provisioned capacity                      | Active capacity consumption       |
| Scaling               | Capacity changes may require modification | Automatic within configured range |
| Development/Test      | Can be used                               | Useful for intermittent workloads |
| Unpredictable Traffic | Capacity planning required                | Strong use case                   |

Easy memory aid:

```text id="1ag9l4"
PROVISIONED
     │
     ▼
Choose Capacity


SERVERLESS
     │
     ▼
Capacity Follows Demand
```

---

# 🎯 When Should I Consider Aurora Serverless?

The lesson highlights:

```text id="1g9e9l"
Variable Workloads

Unpredictable Workloads

E-Commerce Traffic Spikes

Multi-Tenant Applications

Development Environments

Testing Environments
```

The main idea is:

```text id="mb1k4w"
Demand Changes
      │
      ▼
Capacity Changes
      │
      ▼
Pay for Active Usage
```

---

# PART 2: AMAZON AURORA GLOBAL DATABASE

# 🌍 What Is Aurora Global Database?

Aurora Global Database extends an Aurora database across:

```text id="kwnj6z"
Multiple AWS Regions
```

This is useful when applications are distributed across different geographical locations and require low-latency access to database data.

---

# 🏗️ Standard Aurora Architecture

Within a single Region, Aurora can have:

```text id="mk2y10"
Primary Writer

+

Up to 15 Read Replicas
```

Conceptually:

```text id="sj9v4m"
AWS Region
    │
    ▼
Aurora Cluster
    │
    ├── Writer
    ├── Reader 1
    ├── Reader 2
    └── ...
```

---

# 🌎 Global Database Architecture

With Aurora Global Database, Aurora clusters can also exist in other AWS Regions.

Conceptually:

```text id="y4pd3q"
                  Aurora Global Database
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        Primary Region            Secondary Region
              │                         │
              ▼                         ▼
           Writer                    Readers
              │
              ├── Readers
              └── Readers
```

The primary Region contains the writer.

Secondary Regions contain read-only database resources.

---

# ✍️ Primary Region

The primary Region contains:

```text id="1v13cg"
Single Writer
```

and can also contain Aurora read replicas.

Conceptually:

```text id="u3sk16"
Primary Region
      │
      ├── Writer
      ├── Reader
      ├── Reader
      └── Reader
```

The writer handles database modifications.

---

# 📖 Secondary Regions

Other Regions contain secondary clusters used primarily for read operations.

```text id="xqf30v"
Secondary Region
       │
       ├── Reader
       ├── Reader
       └── Reader
```

Applications running closer to those Regions can query the local read replicas.

---

# 🚀 Low-Latency Global Reads

Consider an application deployed in multiple geographical locations.

Without nearby database copies:

```text id="sgw12x"
Application
in Region B
     │
     │ Long Distance
     ▼
Database
in Region A
```

With Aurora Global Database:

```text id="9zxggo"
Application
in Region B
     │
     ▼
Secondary Aurora Cluster
in Region B
```

This can provide lower-latency read access for globally distributed applications.

---

# 🔄 Global Database Replication

The lesson explains that replication between Regions occurs at the:

```text id="zvp8kp"
Storage Layer
```

Conceptually:

```text id="i1z28b"
Primary Region
Aurora Storage
      │
      │ Storage-Level
      │ Replication
      ▼
Secondary Region
Aurora Storage
```

Because replication occurs at the storage layer, the lesson describes changes as being replicated to secondary Regions in approximately:

```text id="gg8d25"
Within 1 Second
```

---

# 🛡️ Disaster Recovery

Aurora Global Database can also support:

```text id="bxclto"
Disaster Recovery

Business Continuity
```

Suppose the primary Region becomes unavailable:

```text id="swc7a8"
Primary Region ❌
      │
      ▼
Secondary Region
      │
      ▼
Promote
      │
      ▼
New Write Capability
```

A secondary cluster in another Region can be promoted, and application traffic can then be redirected to that Region.

---

# 📉 RTO and RPO

The lesson associates the fast storage-level replication of Aurora Global Database with improved:

```text id="et5zvs"
RTO

and

RPO
```

where:

```text id="k9sqha"
RTO
=
Recovery Time Objective


RPO
=
Recovery Point Objective
```

This makes Aurora Global Database useful for workloads requiring cross-Region resilience.

---

# 🌐 Secondary Region Limits

The lesson states that an Aurora Global Database can have:

```text id="5pxmlo"
Up to 5 Secondary Clusters
```

Conceptually:

```text id="zzcq4e"
                 Primary Region
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
 Secondary 1      Secondary 2      Secondary 3
        │              │              │
        ▼              ▼              ▼
    Region B        Region C        Region D
```

with additional secondary clusters possible up to the stated limit.

---

# 📚 Read Replicas in Secondary Clusters

The lesson also describes secondary clusters as supporting multiple read replicas.

Conceptually:

```text id="2izl5q"
Secondary Region
      │
      ▼
Aurora Cluster
      │
      ├── Reader 1
      ├── Reader 2
      ├── Reader 3
      └── ...
```

This provides additional read capacity close to applications running in those Regions.

---

# 🌍 Example Global Application

Consider an application with users across several geographical locations.

```text id="o34f0w"
                       Global Users
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
     Region A            Region B            Region C
        │                   │                   │
        ▼                   ▼                   ▼
 Primary Aurora      Secondary Aurora    Secondary Aurora
     Cluster              Cluster              Cluster
        │                   │                   │
      Writer              Readers             Readers
```

Writes are handled through the primary Region.

Read workloads can be served closer to users from secondary Regions.

---

# 🆚 Aurora Read Replicas vs Aurora Global Database

## Aurora Read Replicas

```text id="z2l2d6"
AWS Region
    │
    ▼
Aurora Cluster
    │
    ├── Writer
    ├── Reader
    └── Reader
```

Useful for:

```text id="2fmnlz"
Read Scaling

High Availability
```

within the Aurora architecture.

---

## Aurora Global Database

```text id="w37g00"
Region A
Primary
   │
   │ Storage Replication
   ▼
Region B
Secondary
   │
   ▼
Readers
```

Useful for:

```text id="pfkqxu"
Global Applications

Low-Latency Global Reads

Cross-Region Disaster Recovery

Business Continuity
```

---

# 🧩 Putting It All Together

Aurora provides several different capabilities depending on the requirement.

```text id="xhzs3m"
                         AMAZON AURORA
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
       Provisioned        Serverless         Global
          Aurora            Aurora          Database
             │                │                │
             ▼                ▼                ▼
       Predetermined      Automatic        Multi-Region
         Capacity          Capacity        Architecture
                            Scaling
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Serverless Means There Is No Database Capacity

Aurora Serverless still requires database compute capacity.

The difference is that the capacity can automatically adjust according to demand.

---

## Mistake 2: Thinking Aurora Serverless Uses Traditional Instance Sizing in the Same Way

Aurora Serverless measures capacity using:

```text id="dfoc2u"
Aurora Capacity Units
        │
        ▼
       ACUs
```

---

## Mistake 3: Confusing Provisioned Aurora with Serverless Aurora

```text id="7cujan"
Provisioned
=
Choose Capacity


Serverless
=
Automatically Adjust Capacity
```

---

## Mistake 4: Confusing Serverless v1 and v2

The lesson emphasizes that Serverless v2 provides additional capabilities such as:

```text id="f3h8hg"
Reader Instances

Multi-AZ

Global Database Support

IAM Authentication

Performance Insights
```

---

## Mistake 5: Thinking Aurora Global Database Means Multiple Writers

The architecture described in the lesson has:

```text id="03qh1r"
Primary Region
     │
     ▼
Writer
```

while secondary Regions contain read replicas.

---

## Mistake 6: Confusing Multi-AZ with Global Database

```text id="rm5o6q"
Multi-AZ
   │
   ▼
Multiple Availability Zones


Global Database
   │
   ▼
Multiple AWS Regions
```

---

## Mistake 7: Thinking Global Replication Happens Through Application-Level Queries

The lesson describes Aurora Global Database replication as occurring at the:

```text id="6v80ve"
Storage Layer
```

---

# ✅ Best Practices

* Consider Aurora Serverless for variable or unpredictable workloads.
* Consider Serverless for development and testing environments with intermittent usage.
* Define appropriate minimum and maximum ACU values for the workload.
* Monitor serverless database capacity and ACU utilization.
* Understand that Serverless v2 can contain writer and reader DB instances.
* Use reader instances when horizontal read scaling is required.
* Use Multi-AZ capabilities when resilience across Availability Zones is required.
* Consider Aurora Global Database when applications operate across multiple Regions.
* Use secondary Regions for low-latency global read access.
* Consider Global Database for cross-Region disaster recovery and business continuity requirements.
* Understand the distinction between scaling capacity and distributing the database globally.

---

# ❓ Interview Questions

### Q1. What is Aurora Serverless?

Aurora Serverless is an Aurora deployment option where database capacity can automatically adjust according to application demand.

---

### Q2. What workloads are good candidates for Aurora Serverless?

The lesson highlights:

```text id="o0d3nl"
Variable Workloads

Unpredictable Workloads

Multi-Tenant Applications

Development

Testing
```

---

### Q3. What is an Aurora Capacity Unit?

An:

```text id="mr2w7j"
Aurora Capacity Unit
        │
        ▼
       ACU
```

represents database capacity.

The lesson describes one ACU as approximately 2 GB of memory with corresponding CPU and networking capability.

---

### Q4. Who manages scaling with Aurora Serverless?

```text id="4z9rqw"
AWS
```

automatically adjusts capacity according to demand within the configured capacity range.

---

### Q5. What capacity values do I configure?

I specify:

```text id="jz0sny"
Minimum ACU

and

Maximum ACU
```

---

### Q6. How is Aurora Serverless billed according to the lesson?

The lesson describes paying for database capacity consumed on a per-second basis.

---

### Q7. What are the two Aurora Serverless versions discussed?

```text id="v6z08g"
Serverless v1

Serverless v2
```

---

### Q8. What happened to Serverless v1 according to the lesson?

The lesson states that Serverless v1 reached end of life on:

```text id="j9grks"
March 31, 2025
```

with migration toward Serverless v2 or provisioned clusters where appropriate.

---

### Q9. Can Serverless v2 have reader DB instances?

Yes.

Serverless v2 supports reader DB instances in addition to the writer.

---

### Q10. Can Serverless v2 operate across multiple Availability Zones?

Yes.

The lesson describes Multi-AZ capabilities for Serverless v2.

---

### Q11. How granular can Serverless v2 scaling be according to the lesson?

```text id="2g8sya"
0.5 ACU
```

increments.

---

### Q12. What is the purpose of the router or proxy fleet?

It sits between application connections and available Serverless database resources and helps maintain connections while capacity changes.

---

### Q13. What is Aurora Global Database?

Aurora Global Database extends an Aurora database across multiple AWS Regions.

---

### Q14. What is the primary use case for Aurora Global Database?

The lesson highlights:

```text id="u2irvq"
Globally Distributed Applications

Low-Latency Reads

Disaster Recovery

Business Continuity
```

---

### Q15. Where is the writer located in an Aurora Global Database?

The writer is located in the:

```text id="upfifb"
Primary Region
```

---

### Q16. What do secondary Regions contain?

They contain secondary Aurora clusters used for read access.

---

### Q17. At what layer does Aurora Global Database replication occur?

```text id="tnlly8"
Storage Layer
```

---

### Q18. How quickly does the lesson describe cross-Region replication?

Approximately:

```text id="0yep48"
Within 1 Second
```

---

### Q19. How many secondary clusters does the lesson say an Aurora Global Database can have?

```text id="hraync"
Up to 5
```

secondary clusters.

---

### Q20. How can Aurora Global Database help with disaster recovery?

If the primary Region becomes unavailable, a secondary cluster in another Region can be promoted and application traffic can be redirected there.

---

### Q21. What is the difference between Aurora Serverless and Aurora Global Database?

```text id="rc12tf"
Aurora Serverless
        │
        ▼
Automatic Capacity Scaling


Aurora Global Database
        │
        ▼
Cross-Region Database Architecture
```

---

# 💡 Key Takeaways

## Aurora Serverless

* Aurora Serverless automatically adjusts database capacity based on workload demand.
* It is useful for variable and unpredictable workloads.
* Multi-tenant, development, and testing environments are also use cases discussed in the lesson.
* Capacity is measured using Aurora Capacity Units (ACUs).
* The lesson describes an ACU as approximately 2 GB of memory with corresponding CPU and networking.
* Minimum and maximum capacity values define the scaling range.
* Billing is based on consumed database capacity.
* The lesson describes the ability to pause capacity by using zero ACU capacity.
* Serverless v2 provides additional capabilities compared with Serverless v1.
* Serverless v2 supports writer and reader instances.
* Serverless v2 supports Multi-AZ configurations.
* Reader instances provide horizontal read scaling.
* A router/proxy fleet helps applications access changing serverless capacity.
* Serverless v2 can scale in increments as small as 0.5 ACU.

## Aurora Global Database

* Aurora Global Database extends Aurora across multiple AWS Regions.
* The primary Region contains the writer.
* Secondary Regions provide read access.
* Cross-Region replication occurs at the storage layer.
* The lesson describes replication as occurring in approximately one second.
* Up to five secondary clusters are described.
* Global Database can provide low-latency reads for globally distributed applications.
* It can also support disaster recovery and business continuity.
* A secondary cluster can be promoted if the primary Region becomes unavailable.

The easiest way to remember the difference is:

```text id="49ggak"
              AMAZON AURORA
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
     SERVERLESS             GLOBAL
          │                   │
          ▼                   ▼
   "How much DB         "Where should
    capacity do          my database
     I need?"             be available?"
          │                   │
          ▼                   ▼
Automatic Scaling       Multiple Regions
```

And the broader Aurora picture becomes:

```text id="4jv88e"
                     AMAZON AURORA
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
       ▼                  ▼                  ▼
   Provisioned        Serverless          Global
     Cluster             v2              Database
       │                  │                  │
       ▼                  ▼                  ▼
Fixed/Selected       Automatic ACU       Multi-Region
 Capacity              Scaling            Access
       │                  │                  │
       ▼                  ▼                  ▼
Writer + Readers    Writer + Readers    Primary Region
                                       +
                                  Secondary Regions
```

---

# 📚 Related Topics

* Amazon Aurora
* Aurora Provisioned Clusters
* Aurora Serverless
* Aurora Serverless v2
* Aurora Capacity Units (ACUs)
* Aurora Reader Instances
* Aurora Multi-AZ
* Aurora Global Database
* Aurora Read Replicas
* Aurora Cluster Endpoints
* Database Auto Scaling
* Cross-Region Replication
* Disaster Recovery
* Business Continuity
* Recovery Time Objective (RTO)
* Recovery Point Objective (RPO)
