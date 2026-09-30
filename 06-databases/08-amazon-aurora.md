# 🌟 Amazon Aurora

> Amazon Aurora is an AWS relational database that is compatible with MySQL and PostgreSQL. It uses a cluster-based architecture designed for high availability, scalability, resilience, and performance.

---

# 📖 Overview

So far, we have looked at Amazon RDS and several relational database engines available through the service, including:

```text
MySQL

PostgreSQL

MariaDB

Microsoft SQL Server

Oracle
```

Another relational database available as part of the RDS offering is:

```text
Amazon Aurora
```

Aurora is different because it is an:

```text
AWS Proprietary
Relational Database
        │
        ▼
Compatible With
        │
   ┌────┴────┐
   ▼         ▼
 MySQL   PostgreSQL
```

This means applications, code, and tools designed to work with MySQL or PostgreSQL can also be used with compatible Aurora databases without requiring major application changes.

---

# 🎯 Why Amazon Aurora?

Aurora uses a different underlying architecture compared with standard RDS database deployments.

It is based on a:

```text
Cluster-Based Architecture
```

designed to provide:

```text
High Availability

Scalability

Resilience

High Performance
```

The lesson describes Aurora as providing up to:

```text
5x MySQL Throughput

3x PostgreSQL Throughput
```

---

# 🏗️ Aurora Cluster Architecture

An Aurora DB cluster contains:

```text
One Primary Instance

+

Zero or More Replica Instances
```

Conceptually:

```text
               Aurora DB Cluster
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
   Primary Instance         Aurora Replicas
   READ + WRITE                  READ
          │                       │
          └───────────┬───────────┘
                      ▼
               Cluster Volume
```

The primary instance performs:

```text
READ

+

WRITE
```

operations.

The replicas can be used for:

```text
READ
```

operations.

---

# ✍️ Aurora Primary Instance

Each Aurora DB cluster has:

```text
One Primary Instance
```

The primary instance performs all data modifications to the cluster volume.

```text
Applications
     │
     │ Read + Write
     ▼
Primary Instance
     │
     ▼
Cluster Volume
```

The primary therefore handles the write operations for the cluster.

---

# 📖 Aurora Read Replicas

Aurora can have:

```text
Up to 15 Read Replicas
```

for the primary instance.

Conceptually:

```text
                    Primary
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       ▼               ▼               ▼
   Replica 1       Replica 2       Replica 3
      READ            READ            READ
```

Additional replicas can be used to increase read capacity.

The lesson contrasts this with the smaller number of replicas available with the standard MySQL and PostgreSQL RDS configurations previously discussed.

---

# 🆚 Aurora Replicas vs Traditional RDS Multi-AZ Standby

This is an important distinction.

In the traditional RDS Multi-AZ architecture discussed earlier:

```text
Primary
   │
   │ Synchronous Replication
   ▼
Standby
```

The primary handles application traffic.

The standby exists primarily for:

```text
High Availability

Failover
```

and is not used for normal application read queries.

---

With Aurora:

```text
Primary
   │
   ├────────► Replica 1
   ├────────► Replica 2
   └────────► Replica 3
```

the replica instances can also serve:

```text
READ Queries
```

Therefore, Aurora replicas can contribute to both the resilience and read scalability of the cluster.

---

# 💾 Aurora Cluster Volume

One of the major architectural differences introduced in the lesson is the:

```text
Aurora Cluster Volume
```

The database instances access this shared cluster storage.

```text
             Primary Instance
                   │
                   │ Read + Write
                   ▼
             Cluster Volume
                   ▲
             ┌─────┴─────┐
             │           │
             │ Read      │ Read
             │           │
         Replica 1   Replica 2
```

The storage layer is therefore separate from the database instances.

---

# ⚡ Aurora Storage

The lesson describes the Aurora cluster volume as using:

```text
SSD Storage
```

providing:

```text
High IOPS

Low Latency
```

The cluster volume can scale to a maximum of:

```text
128 TB
```

of storage.

---

# 🌎 Storage Across Availability Zones

Another important Aurora concept is how its storage is distributed.

The lesson describes Aurora storage as being replicated across:

```text
3 Availability Zones
```

with:

```text
6 Copies
```

of the storage maintained.

Conceptually:

```text
                   Aurora Storage
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
         AZ-A           AZ-B           AZ-C
          │              │              │
       Copy 1         Copy 3         Copy 5
       Copy 2         Copy 4         Copy 6
```

That means:

```text
3 Availability Zones

2 Copies per AZ

6 Storage Copies
```

This distributed storage architecture improves resilience and availability.

---

# 🛡️ Aurora Storage Resilience

Because the data is automatically replicated across multiple Availability Zones:

```text
Database Data
      │
      ▼
Aurora Cluster Volume
      │
      ├── AZ-A
      ├── AZ-B
      └── AZ-C
```

the data remains distributed across the Aurora storage architecture.

This supports faster recovery and failover compared with having to create another copy of the data only after a failure occurs.

---

# 💽 Aurora Storage Configurations

The lesson introduces two storage configuration options:

```text
Aurora I/O-Optimized

Aurora Standard
```

The appropriate choice depends on the application's I/O usage and database spending pattern.

---

# ⚡ Aurora I/O-Optimized

Aurora I/O-Optimized is designed for:

```text
Predictable

I/O-Intensive

Applications
```

The lesson describes this option as charging for:

```text
Database Usage

+

Storage
```

without additional read/write I/O charges.

It is presented as a good option when I/O spending represents approximately:

```text
25% or More
```

of the total Aurora database spend.

---

# 💰 Aurora Standard

Aurora Standard is described as more cost effective for:

```text
Moderate I/O Usage
```

It is presented as a suitable option when I/O spending is:

```text
Less Than 25%
```

of the overall database spend.

---

# 📊 Storage Option Comparison

| Storage Option       | Best Suited For         | I/O Spending Guideline from Lesson |
| -------------------- | ----------------------- | ---------------------------------: |
| Aurora I/O-Optimized | I/O-intensive workloads |                 Around 25% or more |
| Aurora Standard      | Moderate I/O workloads  |                      Less than 25% |

Easy memory aid:

```text
High I/O
   │
   ▼
Aurora I/O-Optimized


Moderate I/O
   │
   ▼
Aurora Standard
```

---

# 🌐 Aurora Endpoints

Because Aurora uses a cluster architecture containing multiple database instances, AWS provides different endpoints for different types of connections.

The lesson introduces:

```text
Cluster Endpoint

Reader Endpoint

Instance Endpoint

Custom Endpoint
```

---

# ✍️ Cluster Endpoint

The:

```text
Cluster Endpoint
```

points to the primary database instance.

Conceptually:

```text
Application
     │
     ▼
Cluster Endpoint
     │
     ▼
Primary Instance
```

This endpoint is used when applications need to perform:

```text
WRITE Operations
```

The primary can also perform reads.

---

# 📖 Reader Endpoint

The:

```text
Reader Endpoint
```

is used for read workloads.

Conceptually:

```text
Read Application
       │
       ▼
 Reader Endpoint
       │
       ▼
 Load Balanced
       │
   ┌───┼───┐
   ▼   ▼   ▼
  R1  R2  R3
```

The reader endpoint can distribute connections across the Aurora read replicas.

This provides:

```text
Read Scalability

+

Improved Read Performance
```

---

# 🎯 Instance Endpoint

An:

```text
Instance Endpoint
```

allows an application or administrator to connect to a specific DB instance.

Conceptually:

```text
Administrator
      │
      ▼
Instance Endpoint
      │
      ▼
Specific Aurora Instance
```

This can be useful when a connection needs to target one particular instance.

---

# 🧩 Custom Endpoint

Aurora also provides:

```text
Custom Endpoints
```

A custom endpoint allows connections to target a:

```text
Subset of DB Instances
```

within the cluster.

For example:

```text
Aurora Cluster
│
├── Primary
│
├── Replica 1
├── Replica 2
├── Replica 3
└── Replica 4
```

Suppose some replicas have different capacities or configurations.

A custom endpoint could target:

```text
Custom Endpoint
      │
      ├── Replica 2
      └── Replica 4
```

rather than every instance in the cluster.

---

# 🎯 Why Use a Custom Endpoint?

The lesson describes custom endpoints as useful when the Aurora cluster contains database instances with different:

```text
Capacities

or

Configurations
```

For example:

```text
Reporting Application
        │
        ▼
Custom Endpoint
        │
        ▼
High-Capacity Replicas
```

while other applications could use different instances.

---

# 🔢 Custom Endpoint Limit

The lesson states that I can create:

```text
Up to 5 Custom Endpoints
```

for each:

```text
Provisioned Aurora Cluster

or

Aurora Serverless v2 Cluster
```

Aurora Serverless is covered separately in a later lesson.

---

# 🗺️ Endpoint Summary

```text
                     Aurora Cluster
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
 Cluster Endpoint    Reader Endpoint   Custom Endpoint
        │                 │                 │
        ▼                 ▼                 ▼
     Primary           Replicas        Selected Instances


Instance Endpoint
        │
        ▼
Specific Instance
```

---

# 💾 Aurora Backups

Aurora automatically backs up the:

```text
Cluster Volume
```

The backup retention period can be configured from:

```text
1 to 35 Days
```

Conceptually:

```text
Aurora Cluster
      │
      ▼
Automatic Backup
      │
      ▼
Retention Period
      │
      ▼
1 - 35 Days
```

---

# 🔄 Restoring an Aurora Backup

When an Aurora backup is restored, the lesson explains that the restore creates:

```text
A New Aurora Cluster
```

Conceptually:

```text
Existing Aurora Cluster
         │
         ▼
       Backup
         │
         ▼
       Restore
         │
         ▼
New Aurora Cluster
```

This is similar to the restore behavior discussed previously for RDS.

---

# 🗄️ AWS Backup Integration

Aurora backups can also be managed using:

```text
AWS Backup
```

This provides a more centralized way to manage:

```text
Backup Policies

Retention Periods

Database Backups
```

Conceptually:

```text
AWS Backup
    │
    ▼
Backup Policies
    │
    ▼
Aurora DB Clusters
```

---

# ⏪ Aurora Backtrack

Another feature introduced in the lesson is:

```text
Aurora Backtrack
```

Backtrack allows an Aurora DB cluster to be rewound to an earlier point in time.

For example:

```text
10:00 AM
Database Healthy
     │
     ▼
11:00 AM
Data Corruption
     │
     ▼
Backtrack
     │
     ▼
Return to Earlier State
```

This can be useful when recovering from database corruption or an unwanted database change.

---

# 🧠 Backtrack Example

Suppose something goes wrong with the database:

```text
08:00 ───── 09:00 ───── 10:00 ───── 11:00
                         │             │
                         │             ▼
                         │        Corruption
                         │
                         ▼
                    Good State
```

With Backtrack:

```text
Current Database
       │
       ▼
Backtrack
       │
       ▼
Earlier Point
```

Instead of performing a traditional restore, the cluster can be rewound to the selected point within the configured backtrack window.

---

# ⚙️ Enabling Backtrack

The lesson emphasizes that Backtrack must be:

```text
Enabled
```

for the cluster.

I also define the:

```text
Backtrack Window
```

which determines how far back the cluster can be rewound.

Conceptually:

```text
Aurora Cluster
      │
      ▼
Enable Backtrack
      │
      ▼
Configure Window
      │
      ▼
Rewind When Required
```

---

# 🆚 Backup Restore vs Backtrack

## Backup Restore

```text
Backup
   │
   ▼
Restore
   │
   ▼
New Cluster
```

## Backtrack

```text
Existing Cluster
      │
      ▼
Rewind
      │
      ▼
Earlier State
```

The important distinction from the lesson is:

```text
Restore
=
Create New Cluster


Backtrack
=
Rewind Cluster
```

---

# 🧬 Aurora Fast Clone

The final feature introduced in this lesson is:

```text
Aurora Fast Clone
```

Fast Clone allows a new Aurora DB cluster to be created using a:

```text
Copy-on-Write
```

approach.

---

# 🐢 Traditional Copy Concept

A traditional full copy could conceptually require:

```text
Source Database
      │
      ▼
Copy All Data
      │
      ▼
New Database
```

This requires another complete copy of the data.

---

# ⚡ Fast Clone Concept

Aurora Fast Clone works differently.

Initially:

```text
               Existing Storage
                     ▲
                     │
              ┌──────┴──────┐
              │             │
              │             │
         Source Cluster   Clone Cluster
```

Both clusters reference the existing data rather than immediately creating a complete duplicate.

---

# ✍️ Copy-on-Write

Additional storage is allocated when data changes.

For example:

```text
Original Data
     │
     ├────────► Source Cluster
     │
     └────────► Clone Cluster
```

If the clone changes some data:

```text
Clone Changes Data
        │
        ▼
Allocate Storage
for Changed Data
```

The unchanged data can continue referencing the original storage.

---

# 💾 Fast Clone Storage Efficiency

Conceptually:

```text
Original Storage
│
├── Data A
├── Data B
├── Data C
└── Data D
```

The clone initially references:

```text
Clone
│
├── Data A ──► Original
├── Data B ──► Original
├── Data C ──► Original
└── Data D ──► Original
```

If Data C changes in the clone:

```text
Clone
│
├── Data A ──► Original
├── Data B ──► Original
├── Data C ──► Clone Storage
└── Data D ──► Original
```

Only changed data requires additional storage.

---

# 🎯 Why Fast Clone?

Because Aurora does not initially make a complete one-for-one copy:

```text
Fast Clone
    │
    ├── Faster Creation
    │
    └── Less Initial Additional Storage
```

This makes cloning an Aurora database more storage efficient.

---

# 🧩 Putting the Architecture Together

```text
                       Applications
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
       Cluster Endpoint             Reader Endpoint
              │                           │
              ▼                           ▼
          Primary                    Read Replicas
       READ + WRITE                  READ ONLY
              │                           │
              └─────────────┬─────────────┘
                            ▼
                     Cluster Volume
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
             AZ-A          AZ-B          AZ-C
              │             │             │
           2 Copies      2 Copies      2 Copies

                    6 Storage Copies
```

---

# 🆚 Standard RDS vs Amazon Aurora

| Feature                            | Standard RDS                               | Amazon Aurora                   |
| ---------------------------------- | ------------------------------------------ | ------------------------------- |
| Database Architecture              | Traditional RDS architecture               | Cluster-based                   |
| Compatibility                      | Depends on selected engine                 | MySQL / PostgreSQL compatible   |
| Primary                            | Read + Write                               | Read + Write                    |
| Read Replicas                      | Supported                                  | Up to 15                        |
| Traditional Multi-AZ Standby Reads | Not accessible for reads                   | Aurora replicas can serve reads |
| Storage                            | Instance/database storage model            | Shared cluster volume           |
| Storage Distribution               | Depends on RDS configuration               | 6 copies across 3 AZs           |
| Maximum Storage in Lesson          | Depends on engine/configuration            | 128 TB                          |
| Reader Endpoint                    | Depends on architecture                    | Yes                             |
| Custom Endpoint                    | Not discussed for standard RDS             | Yes                             |
| Backtrack                          | Not available in standard RDS as described | Supported                       |
| Fast Clone                         | Not discussed                              | Supported                       |

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Aurora Is a Completely Different Database Language

Aurora is compatible with:

```text
MySQL

or

PostgreSQL
```

Existing compatible tools, applications, and code can therefore be used with Aurora.

---

## Mistake 2: Thinking Aurora Uses the Same Architecture as Standard RDS

Aurora uses:

```text
Cluster-Based Architecture
```

with database instances accessing a shared cluster volume.

---

## Mistake 3: Thinking Aurora Replicas Are Like Traditional RDS Multi-AZ Standbys

Traditional RDS Multi-AZ standby:

```text
Standby
   │
   ▼
Failover
```

Aurora replicas:

```text
Aurora Replica
      │
      ▼
Serve Read Queries
```

---

## Mistake 4: Sending Writes Through the Reader Endpoint

Remember:

```text
Cluster Endpoint
      │
      ▼
Primary
      │
      ▼
Writes
```

while:

```text
Reader Endpoint
      │
      ▼
Replicas
      │
      ▼
Reads
```

---

## Mistake 5: Confusing Backtrack with Backup Restore

```text
Backup Restore
=
New Cluster


Backtrack
=
Rewind Existing Cluster
```

---

## Mistake 6: Thinking Fast Clone Immediately Duplicates All Data

Fast Clone uses:

```text
Copy-on-Write
```

and initially references the existing storage.

Additional storage is allocated as data changes.

---

# ✅ Best Practices

* Use the cluster endpoint for workloads that need to write to the database.
* Use the reader endpoint for read workloads that can be distributed across Aurora replicas.
* Use instance endpoints when a specific database instance must be targeted.
* Consider custom endpoints when different groups of instances have different capacities or configurations.
* Understand the difference between Aurora replicas and traditional RDS Multi-AZ standby instances.
* Choose Aurora Standard or Aurora I/O-Optimized based on the workload's I/O characteristics and spending pattern.
* Configure an appropriate backup retention period.
* Consider AWS Backup when centralized backup management is required.
* Enable Backtrack when the ability to rewind the cluster is required.
* Consider Fast Clone when a clone of an Aurora database is needed without immediately duplicating all storage.

---

# ❓ Interview Questions

### Q1. What is Amazon Aurora?

Amazon Aurora is an AWS relational database that is compatible with MySQL and PostgreSQL and uses a cluster-based architecture.

---

### Q2. Is Aurora part of Amazon RDS?

Yes.

The lesson describes Aurora as part of the Amazon RDS offering while also having its own architecture and features.

---

### Q3. Which database engines is Aurora compatible with?

```text
MySQL

PostgreSQL
```

---

### Q4. How many primary instances does an Aurora cluster have?

```text
One Primary Instance
```

The primary handles read and write operations.

---

### Q5. How many Aurora Read Replicas can the cluster have according to the lesson?

```text
Up to 15
```

---

### Q6. Can Aurora replicas serve application read traffic?

Yes.

Aurora replicas can be used for read queries.

---

### Q7. How is this different from the traditional RDS Multi-AZ standby discussed earlier?

The traditional Multi-AZ standby exists primarily for failover and is not used for normal read traffic.

Aurora replicas can serve read traffic.

---

### Q8. What is the Aurora cluster volume?

It is the storage layer used by the Aurora database instances.

The primary performs read and write operations against it, while replicas can read from it.

---

### Q9. Across how many Availability Zones is Aurora storage replicated according to the lesson?

```text
3 Availability Zones
```

---

### Q10. How many storage copies are maintained?

```text
6 Copies
```

with two copies in each of the three Availability Zones.

---

### Q11. What maximum Aurora storage size is mentioned in the lesson?

```text
128 TB
```

---

### Q12. What are the two Aurora storage configurations discussed?

```text
Aurora Standard

Aurora I/O-Optimized
```

---

### Q13. When does the lesson suggest considering Aurora I/O-Optimized?

When I/O spending represents approximately:

```text
25% or More
```

of the overall database spend.

---

### Q14. What is the Aurora cluster endpoint?

It is a DNS endpoint that points to the primary instance and is used for write operations.

---

### Q15. What is the reader endpoint?

It provides access to the read replicas and distributes connections across them for read workloads.

---

### Q16. What is an instance endpoint?

It allows a connection to a specific Aurora DB instance.

---

### Q17. What is a custom endpoint?

It allows applications to connect to a selected subset of DB instances within the cluster.

---

### Q18. How many custom endpoints does the lesson say can be created?

```text
Up to 5
```

for each provisioned Aurora cluster or Aurora Serverless v2 cluster.

---

### Q19. What is the Aurora backup retention period described in the lesson?

```text
1 to 35 Days
```

---

### Q20. What happens when an Aurora backup is restored?

A:

```text
New Aurora Cluster
```

is created.

---

### Q21. What is Aurora Backtrack?

Backtrack allows an Aurora cluster to be rewound to an earlier point within its configured backtrack window.

---

### Q22. What type of problem could Backtrack help recover from?

The lesson gives:

```text
Database Corruption
```

as an example.

---

### Q23. Does Backtrack need to be enabled?

Yes.

The lesson explains that it must be enabled on a per-cluster basis and a backtrack window must be defined.

---

### Q24. What is Aurora Fast Clone?

Fast Clone creates a clone using:

```text
Copy-on-Write
```

instead of initially creating a complete one-for-one copy of all source data.

---

### Q25. Why does Fast Clone require less initial additional storage?

Because the source and clone initially reference the same existing storage.

Additional storage is allocated when data changes.

---

# 💡 Key Takeaways

* Amazon Aurora is an AWS relational database compatible with MySQL and PostgreSQL.
* Aurora uses a cluster-based architecture.
* Each Aurora cluster has one primary instance.
* The primary supports read and write operations.
* Aurora can have up to 15 read replicas.
* Aurora replicas can serve read traffic.
* Aurora uses a separate shared cluster volume.
* The lesson describes up to 128 TB of Aurora storage.
* Aurora maintains six storage copies across three Availability Zones.
* Two storage copies are maintained in each Availability Zone.
* Aurora Standard is intended for moderate I/O workloads.
* Aurora I/O-Optimized is intended for more I/O-intensive workloads.
* The cluster endpoint points to the primary instance.
* The reader endpoint distributes read connections across replicas.
* Instance endpoints target individual database instances.
* Custom endpoints target selected groups of instances.
* The lesson describes up to five custom endpoints.
* Aurora backup retention can be configured from 1 to 35 days.
* Restoring a backup creates a new cluster.
* AWS Backup can be used to centrally manage Aurora backups.
* Aurora Backtrack can rewind a cluster to an earlier point in time.
* Aurora Fast Clone uses copy-on-write to create storage-efficient clones.

The easiest way to visualize Aurora is:

```text
                     AMAZON AURORA
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
           Compute                  Storage
              │                       │
      ┌───────┴───────┐               │
      │               │               ▼
      ▼               ▼         Cluster Volume
   Primary         Replicas             │
READ + WRITE        READ         ┌──────┼──────┐
                                  ▼      ▼      ▼
                                 AZ-A   AZ-B   AZ-C
                                  │      │      │
                                  2      2      2
                                Copies Copies Copies
```

And remember the endpoint model:

```text
CLUSTER ENDPOINT
       │
       ▼
PRIMARY
       │
       ▼
WRITES


READER ENDPOINT
       │
       ▼
READ REPLICAS
       │
       ▼
READS


INSTANCE ENDPOINT
       │
       ▼
SPECIFIC INSTANCE


CUSTOM ENDPOINT
       │
       ▼
SELECTED INSTANCES
```

---

# 📚 Related Topics

* Amazon RDS
* Amazon Aurora
* MySQL
* PostgreSQL
* RDS Multi-AZ
* RDS Read Replicas
* Aurora Replicas
* Aurora Cluster Volume
* Aurora Standard
* Aurora I/O-Optimized
* Aurora Endpoints
* Aurora Backups
* AWS Backup
* Aurora Backtrack
* Aurora Fast Clone
* Aurora Serverless
