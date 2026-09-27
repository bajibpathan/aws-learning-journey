# 📖 Amazon RDS Read Replicas

> Amazon RDS Read Replicas provide read-only copies of a primary database that can be used to offload read queries, increase read capacity, and support additional recovery and cross-Region architectures.

---

# 📖 Overview

In the previous lesson, I learned how:

```text
RDS Multi-AZ
```

provides:

```text
High Availability
       +
Failover
```

A traditional Multi-AZ architecture looks like:

```text
Application
     │
     ▼
Primary RDS
     │
     │ Synchronous Replication
     ▼
Standby RDS
```

The standby exists primarily for:

```text
Failover
```

It does not normally serve application read traffic.

Read Replicas solve a different problem.

They allow me to:

```text
Primary Database
      │
      ├── Write Traffic
      │
      └── Read Traffic
              │
              ▼
       Offload Reads
              │
              ▼
        Read Replicas
```

The primary purpose is:

> **Scale the read capacity of the database and reduce read workload on the primary instance.**

---

# 🎯 Why Do We Need Read Replicas?

Suppose I have a primary RDS database.

```text
Applications
     │
     ▼
Primary RDS
```

The primary is handling:

```text
Writes

+

Reads
```

Now imagine several applications use the database.

```text
                    Primary RDS
                         ▲
                         │
             ┌───────────┼───────────┐
             │           │           │
             │           │           │
        Web App       BI Tool     Reporting
```

The web application may need to:

```text
READ + WRITE
```

But the BI and reporting applications may only need:

```text
READ
```

If every application connects to the primary:

```text
Primary RDS
│
├── Application Reads
├── Application Writes
├── BI Queries
├── Reporting Queries
└── Analytics Queries
```

the primary database must handle all that workload.

Instead, I can create:

```text
Read Replicas
```

and direct read-only applications to them.

---

# 🏗️ Read Replica Architecture

A simple architecture looks like:

```text
                   Application
                       │
                       │ Read + Write
                       ▼
                  Primary RDS
                       │
                       │
               Asynchronous
                Replication
                  ┌────┴────┐
                  │         │
                  ▼         ▼
             Read Replica  Read Replica
                  ▲         ▲
                  │         │
             BI Queries   Reporting
```

The primary continues handling:

```text
READ + WRITE
```

while replicas handle:

```text
READ
```

operations.

---

# 🧠 Primary vs Read Replica

The easiest way to remember the difference is:

```text
PRIMARY
   │
   ▼
READ + WRITE


READ REPLICA
   │
   ▼
READ
```

Read Replicas are not additional writable copies of the same database under normal operation.

---

# ⚡ Read Scaling

The main reason to create Read Replicas is:

```text
Read Scaling
```

Suppose my primary database receives:

```text
10,000 Queries
```

and most of those queries are reads.

Instead of:

```text
All Queries
    │
    ▼
Primary
```

I can distribute read workloads:

```text
                  Primary
                 /       \
                /         \
               ▼           ▼
         Read Replica   Read Replica
```

For example:

```text
Application Writes
        │
        ▼
     Primary


BI Queries
        │
        ▼
 Read Replica 1


Reporting Queries
        │
        ▼
 Read Replica 2
```

This reduces read pressure on the primary database.

---

# 🧪 Example: Business Intelligence Application

Suppose I have:

```text
E-Commerce Application
```

The application needs to:

```text
Create Orders

Update Orders

Read Orders
```

Therefore, it communicates with:

```text
Primary RDS
```

But I also have:

```text
Business Intelligence Application
```

that only needs to analyze existing data.

It does not need to modify the database.

Instead of:

```text
BI Application
      │
      ▼
Primary RDS
```

I can use:

```text
BI Application
      │
      ▼
Read Replica
```

Now:

```text
Primary RDS
     │
     ▼
Less Read Load
```

---

# 🔄 Asynchronous Replication

One of the most important Read Replica concepts is:

```text
Asynchronous Replication
```

The lesson contrasts this with Multi-AZ.

Remember:

```text
Multi-AZ
   │
   ▼
Synchronous Replication


Read Replica
   │
   ▼
Asynchronous Replication
```

---

# 🧠 How Asynchronous Replication Works

With a Read Replica:

```text
Application
     │
     │ Write
     ▼
Primary RDS
     │
     ▼
Data Committed
     │
     ▼
Replicated Later
     │
     ▼
Read Replica
```

The data is first committed on the primary database.

It is then copied to the Read Replica.

Because this replication is asynchronous, the replica may temporarily be behind the primary.

---

# ⏱️ Replication Lag

Because replication is asynchronous, there can be:

```text
Replication Lag
```

For example:

```text
Primary

Order #1001
Order #1002
Order #1003
     │
     │ Replication
     ▼

Read Replica

Order #1001
Order #1002
```

For a short period, the newest data may not yet exist on the Read Replica.

Eventually:

```text
Primary
   │
   ▼
Replication
   │
   ▼
Read Replica

Order #1001
Order #1002
Order #1003
```

This is an important consideration when deciding which applications should read from replicas.

---

# 🆚 Multi-AZ vs Read Replica Replication

This distinction is very important:

| Feature            | Multi-AZ                             | Read Replica            |
| ------------------ | ------------------------------------ | ----------------------- |
| Replication        | Synchronous                          | Asynchronous            |
| Main Purpose       | High Availability                    | Read Scaling            |
| Secondary Readable | No, for traditional Multi-AZ standby | Yes                     |
| Normal Writes      | Primary                              | Primary                 |
| Failover           | Automatic HA architecture            | Replica can be promoted |
| Replication Lag    | Designed for synchronized standby    | Possible                |

Easy memory aid:

```text
MULTI-AZ
   │
   ▼
SYNC
   │
   ▼
HIGH AVAILABILITY


READ REPLICA
   │
   ▼
ASYNC
   │
   ▼
READ SCALING
```

---

# 🌎 Read Replicas Across Availability Zones

Read Replicas can be placed in different Availability Zones.

For example:

```text
                    AWS Region
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
         AZ-A          AZ-B          AZ-C
          │             │             │
          ▼             ▼             ▼
       Primary       Replica 1     Replica 2
```

Applications that only require read access can connect to the replicas.

---

# 🔢 Multiple Read Replicas

The lesson describes standard MySQL and PostgreSQL RDS databases as supporting:

```text
Up to 5 Read Replicas
```

This allows read capacity to be scaled horizontally.

Conceptually:

```text
                   Primary
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
    Replica 1     Replica 2     Replica 3
        ▲             ▲             ▲
        │             │             │
       BI         Reporting      Analytics
```

Instead of making one database increasingly responsible for all reads:

```text
Scale Read Capacity
       │
       ▼
Add Read Replicas
```

---

# 🔗 Read Replica of a Read Replica

The lesson also introduces the ability to create:

```text
Read Replica
     │
     ▼
Read Replica
```

Conceptually:

```text
Primary
   │
   │ Async
   ▼
Replica 1
   │
   │ Async
   ▼
Replica 2
```

However, each additional replication step can introduce more:

```text
Replication Lag
```

For example:

```text
Primary
   │
   ▼
Replica 1
   │
   ▼
Replica 2

Potential Lag
     ↑
```

Therefore, applications using downstream replicas need to account for the possibility that data may not be completely current.

---

# 📈 Scaling Read Capacity

Read Replicas allow me to:

```text
Scale Out
```

read workloads.

For example:

```text
Before

Application
    │
    ▼
Primary
```

As read traffic increases:

```text
After

                   Primary
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
       Read Replica        Read Replica
            ▲                   ▲
            │                   │
       Read Workload        Read Workload
```

This is horizontal read scaling.

---

# ✍️ What About Write Scaling?

The lesson emphasizes that a standard RDS database still has:

```text
One Writable Primary
```

Conceptually:

```text
                Primary
               READ + WRITE
                   │
          ┌────────┴────────┐
          ▼                 ▼
      Replica 1         Replica 2
        READ              READ
```

Read Replicas therefore help with:

```text
Read Scaling
```

not normal write scaling.

More advanced database architectures for distributing writes are outside the scope of this lesson.

---

# 🚀 Promoting a Read Replica

An important capability of Read Replicas is:

```text
Promotion
```

A Read Replica can be promoted to become:

```text
Independent Primary Database
```

Conceptually:

```text
Before

Primary
   │
   │ Replication
   ▼
Read Replica
```

After promotion:

```text
Original Primary        Promoted Replica
      │                       │
      ▼                       ▼
   Database                Database
                              │
                              ▼
                        New Independent
                           Primary
```

---

# 🔌 Replication After Promotion

When a Read Replica is promoted:

```text
Replication Relationship
          │
          ▼
        Broken
```

The promoted database becomes independent.

So:

```text
Before Promotion

Primary
   │
   ▼
Replica


After Promotion

Primary       New Primary
   │              │
   ▼              ▼
Independent    Independent
```

It no longer continues operating as a Read Replica of the original database.

---

# 🆘 Read Replicas and Disaster Recovery

Because a replica contains a copy of the database, it can also help in certain recovery scenarios.

Suppose:

```text
Primary Database
       │
       ▼
Major Failure
```

and a Read Replica is available.

I may be able to:

```text
Read Replica
     │
     ▼
Promote
     │
     ▼
New Primary
```

Instead of restoring an entire database from backup.

---

# ⏱️ Read Replicas and RTO

RTO stands for:

```text
Recovery Time Objective
```

It describes how quickly a workload needs to be restored after a failure.

Compare:

```text
Backup
   │
   ▼
Restore Database
   │
   ▼
Wait for Restore
   │
   ▼
Database Available
```

with:

```text
Read Replica
     │
     ▼
Promote
     │
     ▼
Database Available
```

Because the Read Replica already contains database data, promotion can reduce the time required to make another database available.

Therefore:

```text
Read Replica
     │
     ▼
Potentially Lower RTO
```

---

# 💾 Read Replicas and RPO

The lesson also discusses Read Replicas in relation to recovery capabilities.

RPO stands for:

```text
Recovery Point Objective
```

Read Replicas maintain a continuously replicated copy of database data.

However, because replication is:

```text
Asynchronous
```

I must remember that replication lag can exist.

So a replica should not automatically be treated as a perfect replacement for a backup strategy.

---

# ⚠️ Read Replicas Are Not Backups

This is one of the most important real-world concepts.

Suppose corrupted data is written to the primary:

```text
Primary
   │
   ▼
Corrupted Data
```

Replication continues:

```text
Primary
   │
   │ Async Replication
   ▼
Read Replica
```

Now:

```text
Primary       = Corrupted

Read Replica  = Corrupted
```

The replica has done exactly what it was designed to do:

```text
Replicate Data
```

Unfortunately, that also means replicating bad data.

---

# 💾 Why Backups Are Still Required

Suppose corruption occurred:

```text
Today
  │
  ▼
Bad Data
  │
  ▼
Replicated
```

I might need to restore the database to:

```text
Two Days Ago
```

A Read Replica does not necessarily solve that problem.

A backup can allow me to restore an earlier database state.

Therefore:

```text
Read Replica
      ≠
Backup
```

A good architecture may use both:

```text
Database Protection
│
├── Read Replicas
│      │
│      ├── Read Scaling
│      └── Recovery Options
│
└── Backups
       │
       └── Historical Recovery
```

---

# 🌍 Cross-Region Read Replicas

Another important capability introduced in the lesson is:

```text
Cross-Region Read Replica
```

A Read Replica can exist in another AWS Region.

For example:

```text
Region A
──────────────

Primary RDS
     │
     │
     │ Cross-Region
     │ Replication
     ▼

Region B
──────────────

Read Replica
```

This means a copy of the database can exist geographically separate from the primary database.

---

# 🛡️ Cross-Region Recovery

Consider a regional disaster scenario:

```text
Region A
   │
   ▼
Primary Database ❌
```

If I already have:

```text
Region B
   │
   ▼
Read Replica
```

I could potentially:

```text
Read Replica
     │
     ▼
Promote
     │
     ▼
New Primary
```

This can form part of a disaster recovery strategy.

---

# 🗺️ Cross-Region Architecture

```text
             AWS Region A
                  │
                  ▼
             Primary RDS
                  │
                  │
                  │ Asynchronous
                  │ Replication
                  │
                  ▼
             AWS Region B
                  │
                  ▼
             Read Replica
```

If required:

```text
Read Replica
     │
     ▼
Promote
     │
     ▼
Independent Primary
```

---

# 🆚 Multi-AZ vs Read Replicas

These two concepts are easy to confuse.

## Multi-AZ

```text
Primary
   │
   │ Synchronous
   ▼
Standby
```

Purpose:

```text
High Availability
```

---

## Read Replica

```text
Primary
   │
   │ Asynchronous
   ▼
Read Replica
```

Purpose:

```text
Read Scaling
```

with additional recovery possibilities.

---

# 📊 Quick Comparison

| Feature            | Multi-AZ DB Instance              | Read Replica                     |
| ------------------ | --------------------------------- | -------------------------------- |
| Primary Purpose    | High Availability                 | Read Scaling                     |
| Replication        | Synchronous                       | Asynchronous                     |
| Secondary Readable | No                                | Yes                              |
| Read Traffic       | Primary                           | Can be offloaded                 |
| Write Traffic      | Primary                           | Primary                          |
| Replication Lag    | Not the normal read-replica model | Possible                         |
| Promotion          | Standby during failover           | Replica can be promoted          |
| Cross-Region       | Different concept                 | Supported as described in lesson |
| Backup Replacement | No                                | No                               |

The easiest memory aid:

```text
MULTI-AZ
   │
   ▼
HA


READ REPLICA
   │
   ▼
READ SCALE
```

---

# 🧩 Using Multi-AZ and Read Replicas Together

These technologies solve different problems, so they can exist in the same architecture.

For example:

```text
                         Application
                              │
                              ▼
                          Primary RDS
                         /           \
                        /             \
             Synchronous          Asynchronous
             Replication          Replication
                  │                    │
                  ▼                    ▼
              Standby             Read Replica
                  │                    ▲
                  │                    │
             Failover             BI / Reporting
```

Here:

```text
Standby
   │
   ▼
High Availability
```

while:

```text
Read Replica
   │
   ▼
Read Scaling
```

This is an important architectural distinction.

---

# 🏗️ Example Architecture

Suppose I have:

```text
E-Commerce Application

Business Intelligence Application

Reporting Application
```

I could design:

```text
                         Primary RDS
                        /           \
                       /             \
                      ▼               ▼
               Standby RDS       Read Replica
                                      ▲
                                      │
                           ┌──────────┴──────────┐
                           │                     │
                           ▼                     ▼
                      BI Application       Reporting App
```

The e-commerce application uses the primary for transactions.

The standby supports high availability.

The Read Replica supports read-only workloads.

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Read Replicas Are Multi-AZ Standbys

They are different.

```text
Multi-AZ Standby
       │
       ▼
High Availability


Read Replica
       │
       ▼
Read Scaling
```

---

## Mistake 2: Forgetting the Replication Type

Remember:

```text
Multi-AZ
=
Synchronous


Read Replica
=
Asynchronous
```

---

## Mistake 3: Sending Writes to a Read Replica

A Read Replica is intended for:

```text
READ
```

operations.

Normal writes go to:

```text
Primary
```

---

## Mistake 4: Assuming Read Replicas Always Have the Latest Data

Because replication is asynchronous:

```text
Replication Lag
```

can occur.

Applications reading from a replica need to account for this possibility.

---

## Mistake 5: Thinking Read Replicas Replace Backups

They do not.

If bad data is replicated:

```text
Primary
   │
   ▼
Bad Data
   │
   ▼
Read Replica
```

both copies may contain the same corrupted data.

Backups are still required.

---

## Mistake 6: Forgetting What Happens During Promotion

When a Read Replica is promoted:

```text
Replication Relationship
        │
        ▼
       Ends
```

The promoted database becomes independent.

---

## Mistake 7: Using Read Replicas for Write Scaling

The primary remains the writable database in the architecture covered by this lesson.

Read Replicas scale:

```text
READS
```

not:

```text
WRITES
```

---

# ✅ Best Practices

* Use Read Replicas when the database has significant read traffic.
* Offload reporting and analytics queries from the primary where appropriate.
* Keep application writes directed to the primary database.
* Remember that Read Replica replication is asynchronous.
* Design applications to tolerate replication lag where replicas are used.
* Use Multi-AZ when the primary requirement is high availability.
* Use Read Replicas when the primary requirement is read scaling.
* Use Multi-AZ and Read Replicas together when both HA and read scaling are required.
* Do not treat Read Replicas as replacements for backups.
* Maintain appropriate database backups for historical recovery.
* Consider cross-Region Read Replicas when geographic separation is required.
* Understand the impact before promoting a Read Replica because it becomes an independent database.

---

# ❓ Interview Questions

### Q1. What is an Amazon RDS Read Replica?

A Read Replica is a read-only copy of an RDS primary database that can be used to offload read queries and increase read capacity.

---

### Q2. What is the primary purpose of Read Replicas?

```text
Read Scaling
```

They reduce read workload on the primary database.

---

### Q3. Can applications write to a Read Replica?

No.

Normal writes are performed against the primary database.

---

### Q4. What type of replication is used for Read Replicas?

```text
Asynchronous Replication
```

---

### Q5. What type of replication is used by the traditional Multi-AZ configuration described in the previous lesson?

```text
Synchronous Replication
```

---

### Q6. Why can replication lag occur with Read Replicas?

Because changes are committed to the primary before being asynchronously copied to the replica.

---

### Q7. Give an example of a good workload for a Read Replica.

A:

```text
Business Intelligence

Reporting

Analytics
```

application that only needs to query the database.

---

### Q8. How do Read Replicas improve primary database performance?

Read-only queries can be sent to replicas instead of the primary, reducing the read workload on the primary instance.

---

### Q9. How many Read Replicas does the lesson describe for standard MySQL and PostgreSQL?

```text
Up to 5
```

---

### Q10. Can I create a Read Replica from another Read Replica?

The lesson describes this as possible, but additional replication layers can increase replication lag.

---

### Q11. Can a Read Replica be promoted?

Yes.

A Read Replica can be promoted to become an independent primary database.

---

### Q12. What happens to replication after a Read Replica is promoted?

The replication relationship with the original primary is broken.

---

### Q13. How can a Read Replica help with RTO?

Because the database copy already exists, promoting the replica can be faster than restoring an entire database from backup.

---

### Q14. Does a Read Replica eliminate the need for backups?

No.

Bad or corrupted data can also be replicated to the Read Replica.

---

### Q15. Can a Read Replica exist in another AWS Region?

Yes.

The lesson introduces:

```text
Cross-Region Read Replicas
```

for geographically separated database copies.

---

### Q16. What is the easiest way to distinguish Multi-AZ from Read Replicas?

```text
Multi-AZ
=
High Availability


Read Replica
=
Read Scaling
```

---

### Q17. Can Multi-AZ and Read Replicas be used together?

Yes.

They solve different problems:

```text
Multi-AZ
   │
   ▼
High Availability


Read Replica
   │
   ▼
Read Scaling
```

---

### Q18. Why might a BI application connect to a Read Replica?

Because BI workloads often perform read-heavy queries and may not need to write to the production database.

Offloading those queries reduces load on the primary.

---

# 💡 Key Takeaways

* Amazon RDS Read Replicas are primarily used to scale read capacity.
* The primary database remains the writable copy in the architecture covered by this lesson.
* Read-only applications can connect to Read Replicas instead of the primary.
* This reduces read workload on the primary database.
* Read Replica replication is asynchronous.
* Asynchronous replication can introduce replication lag.
* Traditional Multi-AZ replication is synchronous, while Read Replica replication is asynchronous.
* Multi-AZ primarily addresses high availability.
* Read Replicas primarily address read scaling.
* The lesson describes up to five Read Replicas for standard MySQL and PostgreSQL.
* A Read Replica can itself have another Read Replica as described in the lesson.
* Additional replication layers can increase lag.
* Read Replicas can be promoted to independent primary databases.
* Promotion breaks the replication relationship with the original primary.
* Replica promotion can help reduce recovery time in certain failure scenarios.
* Read Replicas do not replace backups.
* Corrupted data can be replicated from the primary to its replicas.
* Backups are still required for historical recovery.
* Read Replicas can exist in another AWS Region.
* Cross-Region replicas can form part of a disaster recovery strategy.
* Multi-AZ and Read Replicas can be used together because they solve different problems.

The simplest mental model is:

```text
                 PRIMARY RDS
                 READ + WRITE
                      │
                      │
            Asynchronous Replication
                      │
             ┌────────┴────────┐
             ▼                 ▼
       READ REPLICA       READ REPLICA
           READ               READ
             ▲                 ▲
             │                 │
         Reporting         Analytics
```

And the most important comparison is:

```text
MULTI-AZ
   │
   ▼
SYNCHRONOUS
   │
   ▼
HIGH AVAILABILITY


READ REPLICA
   │
   ▼
ASYNCHRONOUS
   │
   ▼
READ SCALING
```

---

# 📚 Related Topics

* Amazon RDS
* RDS Multi-AZ
* Multi-AZ DB Clusters
* RDS Read Scaling
* Synchronous Replication
* Asynchronous Replication
* Replication Lag
* Database Backups
* RDS Automated Backups
* RDS Snapshots
* Recovery Time Objective (RTO)
* Recovery Point Objective (RPO)
* Cross-Region Replication
* Disaster Recovery
* Amazon Aurora
