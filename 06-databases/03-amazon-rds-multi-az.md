# 🛡️ Amazon RDS Multi-AZ

> Amazon RDS Multi-AZ improves database availability by maintaining database instances across multiple Availability Zones so that the database can fail over when the primary instance becomes unavailable.

---

# 📖 Overview

A database is often one of the most critical components of an application.

Consider:

```text
Application
     │
     ▼
RDS Database
```

If the database becomes unavailable:

```text
Application
     │
     ▼
RDS Database ❌
     │
     ▼
Application Impact
```

Amazon RDS provides:

```text
Multi-AZ
```

to improve database availability.

The lesson introduces two Multi-AZ approaches:

```text
Amazon RDS Multi-AZ
│
├── Multi-AZ DB Instance
│
└── Multi-AZ DB Cluster
```

Although both provide high availability, their architecture and capabilities are different.

---

# 🎯 Why Do We Need Multi-AZ?

Imagine I deploy a single RDS database:

```text
AWS Region
│
└── AZ-A
     │
     ▼
 Primary RDS
```

My application connects to that database:

```text
Application
     │
     ▼
Primary RDS
```

If the database instance or Availability Zone experiences a failure:

```text
AZ-A ❌
 │
 ▼
Primary RDS ❌
 │
 ▼
Database Unavailable
```

there is no standby database ready to take over.

Multi-AZ addresses this by maintaining database resources in different Availability Zones.

---

# 🏗️ Two RDS Multi-AZ Options

The course introduces:

| Feature          | Multi-AZ DB Instance | Multi-AZ DB Cluster               |
| ---------------- | -------------------- | --------------------------------- |
| Primary          | 1                    | 1                                 |
| Standby/Readers  | 1 Standby            | 2 Readers                         |
| Standby Readable | No                   | Yes                               |
| Replication      | Synchronous          | Synchronous                       |
| Read Scaling     | No                   | Yes                               |
| Main Purpose     | High Availability    | High Availability + Read Capacity |
| Failover         | Standby promoted     | Reader promoted                   |

The easiest mental model is:

```text
Multi-AZ DB Instance
        =
1 Primary
+
1 Standby


Multi-AZ DB Cluster
        =
1 Writer
+
2 Readers
```

---

# 1. 🛡️ Multi-AZ DB Instance

The first option is the traditional:

```text
Multi-AZ DB Instance
```

Suppose my primary RDS instance is running in:

```text
Availability Zone A
```

When I enable Multi-AZ, RDS creates a standby instance in another Availability Zone.

```text
AWS Region
│
├── AZ-A
│    │
│    ▼
│ Primary RDS
│
└── AZ-B
     │
     ▼
  Standby RDS
```

The two instances are therefore separated across:

```text
Different Availability Zones
           +
Different Subnets
```

---

# 🗃️ Primary Database

Under normal operation, the application communicates with:

```text
Primary RDS
```

The primary database handles:

```text
READS

and

WRITES
```

Conceptually:

```text
Application
     │
     │ Read + Write
     ▼
Primary RDS
```

---

# 💤 Standby Database

The second database acts as:

```text
Standby
```

Conceptually:

```text
Primary
   │
   │ Replication
   ▼
Standby
```

The important point is:

> The standby in a Multi-AZ DB Instance deployment is not used to serve normal application read or write traffic.

So:

```text
Application
     │
     ├────► Primary ✅
     │
     └────► Standby ❌
```

The standby exists primarily for:

```text
High Availability

and

Failover
```

---

# 🔄 Synchronous Replication

The lesson describes Multi-AZ DB Instance replication as:

```text
Synchronous Replication
```

Conceptually:

```text
Application
     │
     ▼
Primary RDS
     │
     │ Synchronous
     │ Replication
     ▼
Standby RDS
```

As database changes are written to the primary storage, they are also replicated to the standby.

This keeps the standby ready to take over if the primary becomes unavailable.

---

# 🌎 Multi-AZ Architecture

A simplified architecture looks like:

```text
                    AWS Region
                        │
                        ▼
                       VPC
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
            AZ-A                  AZ-B
             │                     │
             ▼                     ▼
       DB Subnet A           DB Subnet B
             │                     │
             ▼                     ▼
       Primary RDS ─────────► Standby RDS
                    Sync
                 Replication
```

The primary and standby are located in different Availability Zones.

---

# 🚨 What Can Trigger a Failover?

The lesson introduces several situations that can cause or initiate failover.

Examples include:

```text
Availability Zone Outage

Primary Instance Failure

Manual Failover

Instance Type Changes

Software Patching
```

Conceptually:

```text
Primary RDS
     │
     ▼
Failure / Maintenance
     │
     ▼
Failover
     │
     ▼
Standby Promoted
```

---

# 🔁 Multi-AZ Failover

Suppose the primary database fails:

```text
AZ-A
 │
 ▼
Primary RDS ❌
```

RDS can promote the standby:

```text
AZ-B
 │
 ▼
Standby
 │
 ▼
Promoted
 │
 ▼
New Primary
```

The architecture changes from:

```text
AZ-A                   AZ-B

Primary ────────────► Standby
```

to:

```text
AZ-A                   AZ-B

Unavailable            New Primary
                           │
                           ▼
                       Read + Write
```

---

# 🌐 RDS DNS Endpoint

Applications do not normally connect using the physical address of the underlying database instance.

Instead, they connect using an:

```text
RDS DNS Endpoint
```

Conceptually:

```text
Application
     │
     ▼
RDS Endpoint
     │
     ▼
Primary Database
```

This is important during failover.

---

# 🔄 DNS and Failover

Before failure:

```text
Application
     │
     ▼
DNS Endpoint
     │
     ▼
Primary RDS
```

After failover:

```text
Application
     │
     ▼
Same DNS Endpoint
     │
     ▼
New Primary RDS
```

The application continues using the same database endpoint.

In the background, RDS updates the DNS mapping so that the endpoint resolves to the newly promoted database instance.

Therefore, the application does not need a completely different database endpoint configured manually after failover.

---

# ⏱️ Failover Time

The lesson describes the Multi-AZ DB Instance failover process as typically taking approximately:

```text
60–120 seconds
```

During this period:

```text
Primary Failure
      │
      ▼
Detect Failure
      │
      ▼
Promote Standby
      │
      ▼
Update DNS
      │
      ▼
Application Reconnects
```

Applications should therefore be designed to handle temporary database connectivity interruptions.

---

# 🔁 Restoring Multi-AZ Protection

After failover, the standby has become:

```text
New Primary
```

RDS then restores the Multi-AZ architecture so that the database continues to have standby protection.

Conceptually:

```text
Before

AZ-A                 AZ-B

Primary ──────────► Standby


After Failure

AZ-A                 AZ-B

Failure              New Primary
                         │
                         ▼
                   Standby Recreated
```

This allows the database to continue operating with Multi-AZ protection.

---

# 💾 Multi-AZ and Backups

The lesson also highlights another benefit of Multi-AZ related to database backups.

With a single database instance, backup activity can introduce a brief:

```text
I/O Suspension
```

when the backup process initializes.

Conceptually:

```text
Single RDS
    │
    ▼
Backup Starts
    │
    ▼
Brief I/O Impact
```

With the Multi-AZ configuration described in the lesson, backup operations can make use of the standby copy.

```text
Primary RDS
     │
     │
     ▼
Application Traffic


Standby RDS
     │
     ▼
Backup
```

This helps reduce the backup impact on the primary database workload.

---

# 🧠 Multi-AZ DB Instance Mental Model

The easiest way to remember it is:

```text
PRIMARY
   │
   │ Synchronous Replication
   ▼
STANDBY
```

The primary:

```text
READ + WRITE
```

The standby:

```text
FAILOVER ONLY
```

Under normal operation.

So:

```text
Multi-AZ DB Instance
        =
High Availability
```

not:

```text
Read Scaling
```

---

# 2. 🚀 Multi-AZ DB Cluster

The second architecture introduced in the lesson is:

```text
Multi-AZ DB Cluster
```

This is different from the Multi-AZ DB Instance architecture.

Instead of:

```text
1 Primary
+
1 Standby
```

a Multi-AZ DB Cluster provides:

```text
1 Writer
+
2 Readers
```

distributed across Availability Zones.

---

# 🏗️ Multi-AZ DB Cluster Architecture

Conceptually:

```text
                 Multi-AZ DB Cluster
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
       AZ-A             AZ-B             AZ-C
        │                │                │
        ▼                ▼                ▼
      Writer           Reader           Reader
```

The writer handles:

```text
Read Operations

and

Write Operations
```

while the readers can serve:

```text
Read Operations
```

---

# 🔄 Multi-AZ Cluster Replication

The lesson describes the cluster as using:

```text
Synchronous Replication
```

across the database instances.

Conceptually:

```text
                 Writer
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
       Reader 1          Reader 2
```

Database changes are replicated across the cluster.

The lesson also explains that the database uses transaction logs as part of this replication architecture.

---

# 📖 Readers Are Different from the Traditional Standby

This is one of the most important differences.

With a Multi-AZ DB Instance:

```text
Standby
   │
   ▼
Cannot Normally Serve Reads
```

With a Multi-AZ DB Cluster:

```text
Readers
   │
   ▼
Can Serve Read Traffic
```

Therefore:

```text
Multi-AZ DB Instance

Primary
   │
   └── Standby
         │
         ▼
     Failover


Multi-AZ DB Cluster

Writer
   │
   ├── Reader 1
   └── Reader 2
         │
         ▼
    Read Traffic
```

---

# ⚡ Read Scaling

Suppose my application generates:

```text
20% Writes

80% Reads
```

If everything goes to the writer:

```text
Application
     │
     ▼
Writer
 │
 ├── Reads
 ├── Reads
 ├── Reads
 ├── Reads
 └── Writes
```

the writer handles all database traffic.

With readable instances:

```text
Write Traffic
     │
     ▼
Writer


Read Traffic
     │
     ├────────► Reader 1
     │
     └────────► Reader 2
```

This can reduce the read workload placed on the writer.

---

# 🌐 Multi-AZ DB Cluster Endpoints

The lesson introduces three endpoint concepts:

```text
Multi-AZ DB Cluster
│
├── Cluster Endpoint
├── Reader Endpoint
└── Instance Endpoint
```

Each has a different purpose.

---

# 1. Cluster Endpoint

The:

```text
Cluster Endpoint
```

is used when the application needs to connect to the primary/writer instance.

Conceptually:

```text
Application
     │
     ▼
Cluster Endpoint
     │
     ▼
Writer
```

This is where:

```text
READ + WRITE
```

operations can be performed.

---

# 2. Reader Endpoint

Applications that only need to perform reads can use:

```text
Reader Endpoint
```

Conceptually:

```text
Read-Only Application
        │
        ▼
Reader Endpoint
        │
    ┌───┴───┐
    ▼       ▼
 Reader 1  Reader 2
```

This allows read requests to use the readable instances instead of putting all read workload on the writer.

---

# 3. Instance Endpoint

The lesson also introduces:

```text
Instance Endpoint
```

This allows a connection to a specific database instance.

Conceptually:

```text
Application / Administrator
            │
            ▼
     Instance Endpoint
            │
            ▼
       Specific Instance
```

This can be useful when I need to target a particular member of the cluster.

---

# 🗺️ Endpoint Architecture

Putting the endpoints together:

```text
                    Application
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
      Cluster Endpoint      Reader Endpoint
              │                   │
              ▼             ┌─────┴─────┐
           Writer           ▼           ▼
                         Reader 1     Reader 2
```

And when a specific instance is needed:

```text
Instance Endpoint
       │
       ▼
Specific DB Instance
```

---

# 🚨 Multi-AZ DB Cluster Failover

The readers also provide failover capability.

Suppose:

```text
Writer ❌
```

One of the readers can be promoted:

```text
Writer ❌
    │
    ▼
Reader
    │
    ▼
Promoted
    │
    ▼
New Writer
```

Conceptually:

```text
Before

Writer
  │
  ├── Reader 1
  └── Reader 2


Failure

Writer ❌
  │
  ▼
Reader 1
  │
  ▼
New Writer
```

This provides high availability while also allowing the reader instances to handle read traffic during normal operation.

---

# ⚡ Faster Failover

The lesson highlights that Multi-AZ DB Cluster provides:

```text
Faster Failover
```

than the traditional Multi-AZ DB Instance architecture.

The cluster architecture and transaction-log-based replication help reduce the time required to promote another database instance when the writer becomes unavailable.

---

# 🆚 Multi-AZ DB Instance vs Multi-AZ DB Cluster

| Feature                       | Multi-AZ DB Instance | Multi-AZ DB Cluster                  |
| ----------------------------- | -------------------- | ------------------------------------ |
| Primary/Writer                | 1                    | 1                                    |
| Additional Instances          | 1 Standby            | 2 Readers                            |
| Readable Additional Instances | ❌ No                 | ✅ Yes                                |
| Write Target                  | Primary              | Writer                               |
| Read Scaling                  | ❌ No                 | ✅ Yes                                |
| Replication                   | Synchronous          | Synchronous                          |
| Failover                      | Standby promoted     | Reader promoted                      |
| Main Endpoint                 | DB endpoint          | Cluster endpoint                     |
| Reader Endpoint               | ❌                    | ✅                                    |
| Instance Endpoint             | DB instance endpoint | ✅                                    |
| Main Goal                     | High Availability    | High Availability + Read Performance |

---

# 🎯 When Would I Think About Multi-AZ DB Instance?

When the main requirement is:

```text
Database
   │
   ▼
High Availability
```

and I need:

```text
Primary
   +
Standby
```

for failover.

Conceptually:

```text
Need HA
  │
  ▼
Multi-AZ DB Instance
```

---

# 🎯 When Would I Think About Multi-AZ DB Cluster?

When I need:

```text
High Availability
       +
Readable Instances
       +
Improved Read Performance
       +
Faster Failover
```

Conceptually:

```text
Need HA
  +
Need Read Capacity
  │
  ▼
Multi-AZ DB Cluster
```

---

# 🧠 Important Difference: HA vs Read Scaling

One of the most important concepts from this lesson is:

```text
Multi-AZ DB Instance
        │
        ▼
Standby
        │
        ▼
High Availability
```

versus:

```text
Multi-AZ DB Cluster
        │
        ▼
Readable Instances
        │
        ├── High Availability
        │
        └── Read Capacity
```

Do not assume that the traditional Multi-AZ standby can be used to scale application reads.

---

# 🏗️ Complete Architecture Comparison

## Multi-AZ DB Instance

```text
                       Application
                            │
                            ▼
                       DB Endpoint
                            │
                            ▼
                           VPC
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
            AZ-A                          AZ-B
             │                             │
             ▼                             ▼
          Primary ─────────────────────► Standby
                    Synchronous
                    Replication

             ▲
             │
        Read + Write

Standby:
Failover Only
```

---

## Multi-AZ DB Cluster

```text
                         Application
                              │
                  ┌───────────┴───────────┐
                  │                       │
                  ▼                       ▼
          Cluster Endpoint         Reader Endpoint
                  │                       │
                  ▼                ┌──────┴──────┐
               Writer              ▼             ▼
                 │              Reader 1      Reader 2
                 │                 │             │
                 └─────────────────┴─────────────┘
                      Synchronous Replication
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Multi-AZ Standby Is a Read Replica

For the traditional Multi-AZ DB Instance:

```text
Standby
   │
   ▼
Not Used for
Application Reads
```

It exists for high availability and failover.

---

## Mistake 2: Sending Application Traffic Directly to the Standby

Normal traffic goes to:

```text
Primary
```

not the standby.

---

## Mistake 3: Thinking Multi-AZ Is Only About AZ Failure

Failover can occur for other reasons introduced in the lesson, including:

```text
Instance Failure

Maintenance

Software Patching

Instance Type Changes

Manual Failover
```

---

## Mistake 4: Changing the Application Endpoint During Failover

The application continues to use the same RDS endpoint.

RDS updates the underlying DNS mapping during failover.

---

## Mistake 5: Thinking Multi-AZ DB Instance Improves Read Performance

Traditional Multi-AZ DB Instance primarily provides:

```text
High Availability
```

The standby does not normally serve read traffic.

---

## Mistake 6: Confusing Multi-AZ DB Instance and Multi-AZ DB Cluster

Remember:

```text
DB INSTANCE

1 Primary
+
1 Standby


DB CLUSTER

1 Writer
+
2 Readers
```

---

## Mistake 7: Sending Writes to Reader Instances

Readers are used for:

```text
READ
```

operations.

Writes should go to the writer through the appropriate cluster endpoint.

---

# ✅ Best Practices

* Use Multi-AZ when database availability is important.
* Place database instances across different Availability Zones.
* Design applications to reconnect after temporary database interruptions.
* Use the RDS endpoint rather than hard-coding an underlying database IP address.
* Understand that the traditional Multi-AZ standby does not provide read scaling.
* Use the appropriate endpoint for the workload.
* Send write traffic to the writer/primary.
* Use reader instances for read-heavy workloads when using a Multi-AZ DB Cluster.
* Understand the difference between high availability and read scaling.
* Test database failover behavior before relying on it for production workloads.
* Ensure the application can tolerate the expected failover interruption.

---

# ❓ Interview Questions

### Q1. What is Amazon RDS Multi-AZ?

RDS Multi-AZ improves database availability by maintaining database resources across multiple Availability Zones and providing failover capability.

---

### Q2. What does a traditional Multi-AZ DB Instance deployment contain?

```text
1 Primary

+

1 Standby
```

---

### Q3. Can I read from the standby in a Multi-AZ DB Instance deployment?

No.

The standby is maintained primarily for high availability and failover.

---

### Q4. Can I write to the standby?

No.

Normal read and write operations are performed against the primary.

---

### Q5. How is data replicated between the primary and standby?

The lesson describes:

```text
Synchronous Replication
```

between the primary and standby.

---

### Q6. Why are the primary and standby placed in different Availability Zones?

To reduce the impact of an Availability Zone failure.

---

### Q7. What happens when the primary database fails?

The standby can be promoted to become the new primary.

---

### Q8. Does my application need a completely new database endpoint after failover?

No.

The application continues using the same RDS endpoint while RDS updates the underlying DNS mapping.

---

### Q9. How long does traditional Multi-AZ DB Instance failover take according to this lesson?

Approximately:

```text
60–120 seconds
```

---

### Q10. What are some reasons a failover might occur?

Examples introduced in the lesson include:

```text
AZ Outage

Instance Failure

Manual Failover

Instance Type Changes

Software Patching
```

---

### Q11. Does traditional Multi-AZ improve read scalability?

No.

The standby is not normally available for application read traffic.

---

### Q12. What is a Multi-AZ DB Cluster?

It is a Multi-AZ RDS architecture containing:

```text
1 Writer

+

2 Readers
```

---

### Q13. Can the reader instances serve application reads?

Yes.

That is one of the major differences compared with the standby in a traditional Multi-AZ DB Instance deployment.

---

### Q14. What is the Cluster Endpoint used for?

It connects applications to the writer for read and write operations.

---

### Q15. What is the Reader Endpoint used for?

It provides an endpoint for applications that need to perform read operations using the readable instances.

---

### Q16. What is an Instance Endpoint?

It allows a connection to a specific database instance in the cluster.

---

### Q17. What happens if the writer in a Multi-AZ DB Cluster fails?

One of the readers can be promoted to become the new writer.

---

### Q18. Which option provides read scaling?

```text
Multi-AZ DB Cluster
```

because its additional instances can serve read traffic.

---

### Q19. What is the easiest way to remember the difference?

```text
Multi-AZ DB Instance
=
Primary + Standby
=
High Availability


Multi-AZ DB Cluster
=
Writer + 2 Readers
=
High Availability + Read Capacity
```

---

# 💡 Key Takeaways

* Amazon RDS Multi-AZ is designed to improve database availability.
* A traditional Multi-AZ DB Instance contains one primary and one standby.
* The primary handles normal read and write traffic.
* The standby does not normally serve application read or write traffic.
* Primary and standby are placed in different Availability Zones.
* The lesson describes synchronous replication between primary and standby.
* If the primary fails, the standby can be promoted.
* Applications continue using the same database endpoint after failover.
* RDS changes the underlying DNS mapping during failover.
* Traditional Multi-AZ DB Instance failover is described as taking approximately 60–120 seconds.
* Multi-AZ can also reduce the backup impact on the primary database.
* A Multi-AZ DB Cluster contains one writer and two readable instances.
* The readers can serve application read traffic.
* Multi-AZ DB Clusters therefore provide both high availability and additional read capacity.
* Cluster endpoints are used to reach the writer.
* Reader endpoints are used for read workloads.
* Instance endpoints target specific database instances.
* If the writer fails, a reader can be promoted.
* Multi-AZ DB Cluster provides faster failover than the traditional Multi-AZ DB Instance architecture described in the lesson.
* Multi-AZ and read scaling are related but different concepts.

The simplest mental model is:

```text
MULTI-AZ DB INSTANCE

        Application
             │
             ▼
          Primary
             │
             │ Sync
             ▼
          Standby

Primary = READ + WRITE
Standby = FAILOVER
```

versus:

```text
MULTI-AZ DB CLUSTER

              Writer
             /      \
            /        \
       Reader 1    Reader 2

Writer  = READ + WRITE
Readers = READ
```

And the key distinction:

```text
Need High Availability
        │
        ▼
Multi-AZ DB Instance


Need High Availability
        +
Additional Read Capacity
        │
        ▼
Multi-AZ DB Cluster
```

---

# 📚 Related Topics

* Amazon RDS
* RDS DB Subnet Groups
* RDS Endpoints
* Availability Zones
* High Availability
* Synchronous Replication
* Database Failover
* RDS Automated Backups
* RDS Read Replicas
* Multi-AZ DB Clusters
* Amazon VPC
* Private Subnets
* Amazon Route 53 / DNS Concepts
