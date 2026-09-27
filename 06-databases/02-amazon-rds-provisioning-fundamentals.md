# 🗄️ Amazon RDS Provisioning Fundamentals

> Amazon Relational Database Service (RDS) is a managed relational database service. When provisioning an RDS database, I choose the database engine, DB instance class, storage, networking, and other database configuration while AWS manages the underlying infrastructure.

---

# 📖 Overview

In the previous lesson, I learned the fundamentals of relational databases and received an introduction to:

```text
Amazon RDS
```

Instead of installing and managing a relational database myself on an EC2 instance, I can use Amazon RDS as a managed database service.

At a high level, provisioning an RDS database involves choosing:

```text
Amazon RDS
    │
    ├── Database Engine
    │
    ├── DB Instance Class
    │
    ├── Storage
    │
    └── Network / VPC
```

There are more advanced RDS features, such as:

```text
Multi-AZ

Read Replicas
```

but first I need to understand the basic building blocks.

---

# 🎯 What Is Amazon RDS?

Amazon RDS stands for:

```text
Amazon
Relational
Database
Service
```

It is a managed service for running relational databases on AWS.

Without RDS, I could do something like:

```text
Amazon EC2
    │
    ▼
Operating System
    │
    ▼
Install Database
    │
    ▼
Configure Database
    │
    ▼
Manage Database
```

With RDS:

```text
Amazon RDS
    │
    ▼
Choose Database Configuration
    │
    ▼
AWS Provisions Database
```

AWS manages the underlying infrastructure used by the RDS database.

---

# 🆚 Database on EC2 vs Amazon RDS

## Database on EC2

```text
EC2 Instance
    │
    ▼
Operating System
    │
    ▼
Database Software
```

I manage the operating system and database environment.

---

## Amazon RDS

```text
Amazon RDS
    │
    ▼
Managed DB Instance
    │
    ▼
Database
```

AWS manages the underlying instance and operating system.

With standard Amazon RDS, I do not log in to the underlying operating system.

Instead, I work at the:

```text
Database Level
```

where I can create databases, create tables, query data, and perform CRUD operations.

---

# 🚫 No Operating System Access

This is an important distinction between:

```text
EC2
```

and:

```text
RDS
```

With EC2:

```text
EC2
 │
 ├── OS Access
 │
 └── Database Access
```

With standard RDS:

```text
RDS
 │
 ├── OS Access ❌
 │
 └── Database Access ✅
```

The underlying infrastructure is managed by AWS.

The lesson also mentions:

```text
RDS Custom
```

as an exception that provides additional access, but that is outside the scope of this introductory lesson.

---

# 🏗️ Basic RDS Provisioning Process

At a high level:

```text
Create RDS Database
        │
        ▼
1. Choose Database Engine
        │
        ▼
2. Choose DB Instance Class
        │
        ▼
3. Choose Storage
        │
        ▼
4. Configure VPC Networking
        │
        ▼
5. Deploy Database
```

These are the main concepts I need to understand before deploying my first RDS database.

---

# 1. 🗃️ Choose a Database Engine

The first decision is:

```text
Which relational database engine
do I want to use?
```

The lesson introduces the following RDS database engines:

```text
Amazon RDS
│
├── MySQL
├── PostgreSQL
├── MariaDB
├── Microsoft SQL Server
├── IBM Db2
├── Oracle Database
└── Amazon Aurora
```

The engine determines the database technology my application will use.

---

# 🌟 Amazon Aurora

Amazon Aurora is AWS's relational database offering that is compatible with:

```text
MySQL

and

PostgreSQL
```

Conceptually:

```text
             Amazon Aurora
                  │
          ┌───────┴───────┐
          ▼               ▼
       MySQL          PostgreSQL
     Compatible        Compatible
```

Aurora provides additional capabilities that will be covered separately.

For now, I only need to recognize it as another relational database option available through Amazon RDS.

---

# 2. 🖥️ Choose a DB Instance Class

After selecting the database engine, I need to decide:

```text
How much compute capacity
does my database need?
```

An RDS database runs using underlying compute resources managed by AWS.

I choose a:

```text
DB Instance Class
```

that determines resources such as:

```text
vCPU

Memory

Network Capacity
```

---

# 🧠 DB Instance Classes

This concept is similar to choosing an EC2 instance type.

With EC2:

```text
EC2 Instance Type
       │
       ├── vCPU
       ├── Memory
       └── Network
```

With RDS:

```text
DB Instance Class
       │
       ├── vCPU
       ├── Memory
       └── Network
```

The major difference is that the underlying RDS instance is managed by AWS.

---

# 📌 Example DB Instance Class

The lesson uses an example similar to:

```text
db.m7g.large
```

The name identifies a particular database instance class and size.

Conceptually:

```text
db.m7g.large
│   │     │
│   │     └── Size
│   │
│   └── Instance Family
│
└── RDS DB Instance
```

Different instance classes provide different amounts of:

```text
CPU

RAM

Networking
```

---

# ⚖️ Choosing the Right Size

Larger database instances provide more compute resources.

Conceptually:

```text
Small DB Instance
      │
      ▼
Less CPU / Memory
      │
      ▼
Lower Cost
```

versus:

```text
Large DB Instance
      │
      ▼
More CPU / Memory
      │
      ▼
Higher Cost
```

Therefore:

> Bigger is not automatically better.

I should choose a DB instance class based on the requirements of the database workload.

---

# 🏷️ DB Instance Categories

The available DB instance classes depend on the database engine.

The lesson introduces categories such as:

```text
Standard

Memory Optimized

Burstable
```

For example:

```text
Database Workload
      │
      ├── General Purpose
      │       ▼
      │    Standard
      │
      ├── Memory Intensive
      │       ▼
      │ Memory Optimized
      │
      └── Smaller / Variable Workload
              ▼
           Burstable
```

Not every database engine necessarily supports exactly the same instance classes.

---

# ⚡ Burstable DB Instances

Burstable DB instances are similar in concept to burstable EC2 instance families.

They can be useful when workloads do not require consistently high CPU performance.

Conceptually:

```text
Normal Workload
      │
      ▼
Lower CPU Usage
      │
      ▼
Occasional Burst
      │
      ▼
Higher CPU Requirement
```

They can provide a lower-cost option for suitable workloads.

---

# ☁️ Aurora Serverless

The lesson also briefly introduces:

```text
Aurora Serverless
```

as another option available with Amazon Aurora.

The detailed behavior of Aurora Serverless is outside the scope of this lesson.

For now, I only need to recognize:

```text
Amazon Aurora
      │
      └── Serverless Option
```

---

# 3. 💾 Choose Database Storage

Compute and storage are separate decisions.

This is important.

```text
RDS Database
     │
     ├── Compute
     │
     │     ▼
     │ DB Instance Class
     │
     └── Storage
           ▼
       Storage Type
```

Therefore, choosing a larger DB instance does not automatically mean I have selected the required database storage configuration.

---

# 💽 RDS Storage Options

The lesson introduces storage choices such as:

```text
SSD Storage

Provisioned IOPS Storage
```

Some combinations may also provide access to:

```text
Magnetic Storage
```

but the lesson recommends using the newer storage options instead.

---

# ⚡ SSD Storage

SSD-based storage can be used for general database workloads.

Conceptually:

```text
RDS
 │
 ▼
SSD Storage
 │
 ▼
General Database Workloads
```

---

# 🚀 Provisioned IOPS

For workloads that require more predictable storage performance, I can use:

```text
Provisioned IOPS
```

Conceptually:

```text
High I/O Requirement
        │
        ▼
Predictable IOPS Requirement
        │
        ▼
Provisioned IOPS
```

This can be useful for database workloads where storage performance is particularly important.

---

# 🧩 Compute and Storage Are Decoupled

This is an important concept to remember:

```text
DB Instance Class
       ≠
Storage Type
```

For example:

```text
RDS Database
│
├── Compute
│     └── db.m7g.large
│
└── Storage
      └── SSD
```

I choose both according to my workload requirements.

---

# 🔄 RDS and OLTP Workloads

Amazon RDS is designed for relational database workloads such as:

```text
Online Transaction Processing
```

or:

```text
OLTP
```

Typical examples include:

```text
E-Commerce Applications

Transactional Applications

Customer Systems

Order Processing Systems
```

For example:

```text
Customer
    │
    ▼
Places Order
    │
    ▼
Application
    │
    ▼
RDS Database
    │
    ├── Customer Table
    ├── Order Table
    ├── Product Table
    └── Payment Table
```

These workloads frequently:

```text
Create Data

Read Data

Update Data

Delete Data
```

which connects back to the:

```text
CRUD
```

concept from the previous lesson.

---

# 🔗 Complex Relationships and Queries

RDS is also appropriate when applications need relational database capabilities such as:

```text
Multiple Tables

Relationships

Primary Keys

Foreign Keys

Joins

Complex Queries
```

For example:

```text
Customers
    │
    ▼
Orders
    │
    ▼
Order Items
    │
    ▼
Products
```

SQL can then be used to combine and query these related tables.

---

# 4. 🌐 RDS Networking

One of the most important concepts when deploying an RDS database is:

```text
Amazon RDS
     │
     ▼
Amazon VPC
```

An RDS DB instance is deployed within a VPC.

More specifically, RDS needs to know:

```text
Which subnets can be used
for the database?
```

This introduces another important concept:

```text
DB Subnet Group
```

---

# 🏗️ What Is a DB Subnet Group?

A DB Subnet Group is a collection of subnets that Amazon RDS can use when deploying database resources.

Conceptually:

```text
VPC
│
├── AZ-A
│    └── Private Subnet A
│
└── AZ-B
     └── Private Subnet B
```

These subnets can be grouped into:

```text
DB Subnet Group
│
├── Private Subnet A
└── Private Subnet B
```

Amazon RDS can then use the subnets defined in that DB Subnet Group.

---

# 🎯 Why Does RDS Need a DB Subnet Group?

When I configure a DB Subnet Group, I am effectively telling RDS:

> These are the subnets that can be used for my database deployment.

Conceptually:

```text
DB Subnet Group
       │
       ├── Subnet A
       └── Subnet B
             │
             ▼
         Amazon RDS
```

The lesson emphasizes using subnets across:

```text
Multiple Availability Zones
```

For example:

```text
VPC
│
├── Availability Zone A
│    │
│    └── Private Subnet A
│
└── Availability Zone B
     │
     └── Private Subnet B
```

Then:

```text
DB Subnet Group
│
├── Private Subnet A
└── Private Subnet B
```

---

# 🔒 Place Databases in Private Subnets

The architecture shown in the lesson places the database in:

```text
Private Subnets
```

rather than directly exposing it to the internet.

A common application architecture looks like:

```text
Internet
    │
    ▼
Application
    │
    ▼
Private Database
```

or:

```text
VPC
│
├── Application Layer
│
│       │
││       ▼
│
└── Private Database Layer
        │
        ▼
      Amazon RDS
```

The database should normally be accessed by the application rather than directly by internet users.

---

# 🏗️ Basic RDS VPC Architecture

A simple architecture from this lesson looks like:

```text
                VPC
                 │
       ┌─────────┴─────────┐
       │                   │
       ▼                   ▼
      AZ-A                AZ-B
       │                   │
       ▼                   ▼
Private Subnet A      Private Subnet B
       │                   │
       └─────────┬─────────┘
                 │
                 ▼
          DB Subnet Group
                 │
                 ▼
             Amazon RDS
```

The DB Subnet Group gives RDS subnet choices across multiple Availability Zones.

---

# 🛡️ Why Multiple Availability Zones?

Using subnets across multiple Availability Zones prepares the architecture for features such as:

```text
Multi-AZ
```

The lesson only introduces Multi-AZ at a high level.

The detailed configuration comes later.

---

# 🏢 Single Database Problem

Suppose I only have:

```text
AZ-A
 │
 ▼
Primary RDS
```

If something happens to:

```text
Database Instance

or

Availability Zone
```

my database could become unavailable.

Conceptually:

```text
AZ-A
 │
 ▼
Primary DB ❌
 │
 ▼
Application Cannot
Access Database
```

This creates a potential availability problem.

---

# 🛡️ Multi-AZ Concept

Multi-AZ addresses this by providing:

```text
Primary Database
       +
Standby Database
```

across Availability Zones.

Conceptually:

```text
                VPC
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
       AZ-A              AZ-B
        │                 │
        ▼                 ▼
    Primary DB         Standby DB
        │                 ▲
        │                 │
        └── Replication ──┘
```

The lesson describes this replication as:

```text
Synchronous Replication
```

---

# 🔄 Multi-AZ Failure Scenario

Under normal operation:

```text
Application
     │
     ▼
Primary RDS
     │
     ▼
Standby Copy
```

If the primary database or its Availability Zone experiences a failure:

```text
Primary RDS ❌
      │
      ▼
Standby Database
      │
      ▼
Promoted
      │
      ▼
New Primary
```

This helps improve database availability.

The detailed behavior of Multi-AZ will be covered separately.

---

# 🧠 Why the DB Subnet Group Matters

Now the purpose of the DB Subnet Group becomes clearer.

```text
DB Subnet Group
│
├── Subnet in AZ-A
│
└── Subnet in AZ-B
```

provides RDS with subnet options across Availability Zones.

This supports architectures such as:

```text
AZ-A
Primary Database

      +

AZ-B
Standby Database
```

---

# 🔮 Advanced RDS Features Coming Later

The lesson introduces but does not yet explore:

```text
Multi-AZ

Read Replicas

Aurora

Aurora Serverless

RDS Custom
```

These should be treated as separate concepts.

For now, the important goal is understanding:

```text
How an RDS database
is provisioned.
```

---

# 🧩 Putting Everything Together

When I provision an RDS database, the high-level process is:

```text
Need Relational Database
          │
          ▼
Choose Database Engine
          │
          ▼
Choose DB Instance Class
          │
          ▼
Choose Storage
          │
          ▼
Choose VPC
          │
          ▼
Configure DB Subnet Group
          │
          ▼
Deploy RDS Database
```

---

# 🏗️ Complete Architecture

```text
                        AWS Region
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
     Private Subnet A              Private Subnet B
             │                             │
             └──────────────┬──────────────┘
                            │
                            ▼
                     DB Subnet Group
                            │
                            ▼
                       Amazon RDS
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
          Database Engine          DB Instance
                                      Class
                │                       │
                └───────────┬───────────┘
                            │
                            ▼
                          Storage
```

With Multi-AZ:

```text
                     DB Subnet Group
                            │
               ┌────────────┴────────────┐
               │                         │
               ▼                         ▼
              AZ-A                      AZ-B
               │                         │
               ▼                         ▼
          Primary RDS              Standby RDS
               │                         ▲
               │                         │
               └── Synchronous ──────────┘
                   Replication
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking RDS Runs Without Compute

RDS is managed, but the database still requires compute resources.

I select:

```text
DB Instance Class
```

to define the compute capacity.

---

## Mistake 2: Thinking I Can SSH into Standard RDS

With standard RDS:

```text
Operating System Access ❌

Database Access ✅
```

AWS manages the underlying operating system.

---

## Mistake 3: Thinking the Database Engine Determines Everything

Choosing:

```text
MySQL
```

is only one part of the configuration.

I still need to consider:

```text
DB Instance Class

Storage

Networking
```

---

## Mistake 4: Thinking Compute and Storage Are the Same Configuration

They are separate decisions.

```text
Compute
   │
   ▼
DB Instance Class


Storage
   │
   ▼
Storage Configuration
```

---

## Mistake 5: Forgetting the DB Subnet Group

RDS needs a DB Subnet Group that identifies the subnets available for database deployment.

---

## Mistake 6: Putting the Database Directly on the Internet

The architecture presented in the lesson uses:

```text
Private Subnets
```

for the database layer.

The application should communicate with the database through the VPC rather than unnecessarily exposing the database directly to the internet.

---

## Mistake 7: Confusing Multi-AZ with the Basic RDS Instance Configuration

Multi-AZ is an additional availability capability.

First understand:

```text
Engine

Instance Class

Storage

VPC

DB Subnet Group
```

Then build on those concepts with Multi-AZ.

---

# ✅ Best Practices

* Choose the database engine based on application requirements.
* Choose an appropriate DB instance class instead of automatically selecting the largest option.
* Understand that supported DB instance classes can vary by database engine.
* Select storage according to the database I/O requirements.
* Treat compute and storage as separate configuration decisions.
* Deploy the database within a VPC.
* Use a DB Subnet Group containing subnets across multiple Availability Zones.
* Use private subnets for the database architecture presented in this lesson.
* Avoid unnecessary direct internet exposure of database resources.
* Understand the basic RDS provisioning model before moving to advanced features.
* Consider Multi-AZ when database availability is important.

---

# ❓ Interview Questions

### Q1. What is Amazon RDS?

Amazon RDS is a managed relational database service on AWS.

---

### Q2. Do I need to install the database manually on an EC2 instance when using RDS?

No.

AWS provisions and manages the underlying database infrastructure.

---

### Q3. Can I access the operating system of a standard RDS DB instance?

No.

With standard RDS, AWS manages the underlying operating system.

---

### Q4. What are the main decisions when provisioning an RDS database?

At a high level:

```text
Database Engine

DB Instance Class

Storage

VPC / Networking
```

---

### Q5. What database engines are introduced in this lesson?

```text
MySQL

PostgreSQL

MariaDB

Microsoft SQL Server

IBM Db2

Oracle Database

Amazon Aurora
```

---

### Q6. What is a DB instance class?

It defines the compute resources used by the RDS database, such as CPU, memory, and networking capability.

---

### Q7. Is the DB instance class similar to an EC2 instance type?

Conceptually, yes.

Both determine compute resources, but the underlying RDS infrastructure is managed by AWS.

---

### Q8. Does every RDS engine support exactly the same DB instance classes?

No.

Available DB instance classes depend on the database engine.

---

### Q9. Are compute and storage coupled in RDS?

No.

I select the DB instance class and storage configuration separately.

---

### Q10. What is OLTP?

OLTP stands for:

```text
Online Transaction Processing
```

It describes transactional workloads such as order processing and e-commerce applications.

---

### Q11. What is a DB Subnet Group?

A DB Subnet Group is a collection of subnets that Amazon RDS can use for database deployment.

---

### Q12. Why should a DB Subnet Group contain subnets across multiple Availability Zones?

It provides RDS with subnet options across AZs and supports availability designs such as Multi-AZ.

---

### Q13. Where should the database normally be placed in the architecture presented here?

In:

```text
Private Subnets
```

inside the VPC.

---

### Q14. What is Multi-AZ at a high level?

Multi-AZ provides a primary database and a standby database across Availability Zones to improve availability.

---

### Q15. How is data replicated to the standby in the Multi-AZ concept introduced here?

Using:

```text
Synchronous Replication
```

---

### Q16. What happens if the primary database fails in the Multi-AZ architecture?

The standby can be promoted to take over as the primary database.

---

### Q17. Is Multi-AZ covered in detail in this lesson?

No.

It is introduced at a high level and will be covered separately.

---

# 💡 Key Takeaways

* Amazon RDS is a managed relational database service.
* I do not install the database manually on an EC2 instance when using standard RDS.
* AWS manages the underlying RDS infrastructure and operating system.
* I normally interact with RDS at the database level rather than the operating-system level.
* The first major decision is selecting the database engine.
* The lesson introduces MySQL, PostgreSQL, MariaDB, Microsoft SQL Server, IBM Db2, Oracle Database, and Amazon Aurora.
* A DB instance class determines the database compute capacity.
* Different database engines can support different DB instance classes.
* Standard, memory-optimized, and burstable instance categories are introduced.
* Compute and storage are separate configuration decisions.
* SSD and Provisioned IOPS storage are introduced as storage choices.
* RDS is suitable for relational and OLTP workloads.
* RDS DB instances are deployed within a VPC.
* A DB Subnet Group defines the subnets RDS can use.
* The lesson's architecture uses private subnets across multiple Availability Zones.
* Multi-AZ introduces a standby database in another Availability Zone.
* The lesson describes synchronous replication between the primary and standby.
* If the primary fails, the standby can be promoted.
* Multi-AZ and Read Replicas are advanced RDS topics that will be covered separately.

The simplest mental model is:

```text
AMAZON RDS
     │
     ▼
Choose Engine
     │
     ▼
Choose Compute
     │
     ▼
Choose Storage
     │
     ▼
Choose Network
     │
     ▼
DB Subnet Group
     │
     ▼
Deploy Database
```

And for availability:

```text
DB Subnet Group
      │
      ├── AZ-A
      │     │
      │     ▼
      │  Primary
      │
      └── AZ-B
            │
            ▼
         Standby
```

---

# 📚 Related Topics

* Relational Databases
* Amazon RDS
* Amazon Aurora
* MySQL
* PostgreSQL
* MariaDB
* Microsoft SQL Server
* Oracle Database
* IBM Db2
* RDS DB Instance Classes
* RDS Storage
* Amazon VPC
* Private Subnets
* DB Subnet Groups
* RDS Multi-AZ
* RDS Read Replicas
* Online Transaction Processing (OLTP)
