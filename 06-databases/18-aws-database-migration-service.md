# 🚚 AWS Database Migration Service (DMS)

> AWS Database Migration Service (AWS DMS) is a fully managed service for migrating databases and other supported data stores to AWS. It supports one-time migrations, ongoing replication using Change Data Capture (CDC), homogeneous and heterogeneous migrations, and targets such as Amazon RDS, DynamoDB, Redshift, and Amazon S3.

---

# 📖 Overview

Organizations often need to migrate databases from:

- On-premises environments
- Databases running on Amazon EC2
- Existing database platforms

to AWS-managed database services.

AWS provides:

```text
AWS Database Migration Service
             │
             ▼
            DMS
```

DMS can help migrate relational databases, data warehouses, NoSQL databases, and other supported data stores.

Typical migration:

```text
Self-Managed Database
        │
        ▼
      AWS DMS
        │
        ▼
AWS Managed Database
```

Examples of AWS targets discussed in the lesson include:

```text
Amazon RDS

Amazon DynamoDB

Amazon Redshift

Amazon S3
```

---

# 🎯 Why AWS DMS?

Without a migration service, moving a production database can involve:

```text
Export Data
    │
    ▼
Transfer Data
    │
    ▼
Import Data
    │
    ▼
Track New Changes
    │
    ▼
Synchronize Databases
    │
    ▼
Cut Over Application
```

AWS DMS helps manage this migration and replication process.

The service can move existing data and can also continue capturing changes occurring while the migration is underway.

---

# 🏗️ Core DMS Architecture

A basic DMS architecture contains:

```text
Source Database

Replication Instance

Source Endpoint

Target Endpoint

Target Database

Migration Task
```

Conceptually:

```text
┌───────────────────┐
│  Source Database  │
│                   │
│ On-Premises / EC2 │
└─────────┬─────────┘
          │
          │ Source Endpoint
          ▼
┌───────────────────────┐
│  Replication Instance │
│                       │
│       AWS DMS         │
└─────────┬─────────────┘
          │
          │ Target Endpoint
          ▼
┌───────────────────────┐
│    Target Database    │
│                       │
│ RDS / DynamoDB / etc. │
└───────────────────────┘
```

The replication instance performs the data replication between the source and destination.

---

# 🗄️ Source Database

The source database can be located:

```text
On-Premises

or

Amazon EC2
```

The lesson mentions database engines and platforms such as:

- MySQL
- PostgreSQL
- Microsoft SQL Server
- Oracle
- Teradata
- NoSQL databases

If the source database is on-premises, connectivity must exist between the on-premises environment and the AWS environment.

---

# 🎯 Target Database

The target is the destination for the migrated data.

Examples discussed include:

```text
Amazon RDS

Amazon DynamoDB

Amazon Redshift
```

Later in the lecture, Amazon S3 is also introduced as a DMS target.

---

# 🔗 Source and Target Endpoints

DMS uses endpoints to identify the source and destination.

```text
Source Endpoint
      │
      ▼
Source Database


Target Endpoint
      │
      ▼
Target Database
```

The source endpoint contains the information required to connect to the source.

The target endpoint identifies where DMS should send the migrated data.

---

# ⚙️ Replication Instance

The lesson introduces the:

```text
Replication Instance
```

as the component performing the replication between the source and target.

The basic flow is:

```text
Source
   │
   ▼
Source Endpoint
   │
   ▼
Replication Instance
   │
   ▼
Target Endpoint
   │
   ▼
Target
```

Migration tasks run through this configuration to move and replicate data.

---

# 🔄 How DMS Migration Works

The lesson explains the migration process in three main phases:

```text
1. Full Load

2. Apply Cached Changes

3. Ongoing Replication
```

---

# 1️⃣ Phase 1: Full Load

During:

```text
FULL LOAD
```

DMS copies the existing data from the source database to the target.

```text
Source Database
      │
      │ Existing Data
      ▼
    AWS DMS
      │
      ▼
Target Database
```

DMS loads the source tables into the target data store.

---

# 📝 Changes During Full Load

The source database may still be receiving transactions while the full load is happening.

For example:

```text
DMS Copying Table
       │
       │
Application
       │
       ▼
INSERT / UPDATE / DELETE
```

DMS needs to account for these changes.

The lesson explains that changes occurring after the full load of a particular table begins are captured and temporarily cached.

```text
Full Load Running
      │
      ├── Copy Existing Data
      │
      └── Cache New Changes
```

The point at which change capture begins can therefore differ for each table.

---

# 2️⃣ Phase 2: Apply Cached Changes

Once the full load for a table completes:

```text
Full Load Complete
       │
       ▼
Apply Cached Changes
```

DMS applies the changes that occurred while the initial data was being loaded.

Conceptually:

```text
Original Data
     │
     ▼
Full Load
     │
     ▼
Cached Changes
     │
     ▼
Apply Changes
     │
     ▼
Target Updated
```

This helps bring the target closer to the current state of the source.

---

# 3️⃣ Phase 3: Ongoing Replication

After the full load and cached changes have been applied, DMS can continue capturing new transactions.

This is:

```text
Ongoing Replication

or

Change Data Capture (CDC)
```

Architecture:

```text
Source Database
      │
      │ New Changes
      ▼
    AWS DMS
      │
      ▼
Target Database
```

The target continues receiving changes occurring on the source.

---

# ⏱️ Replication Lag

At the beginning of ongoing replication, there may be a backlog of transactions.

This can create:

```text
Source
  │
  ▼
Transaction Backlog
  │
  ▼
Replication Lag
  │
  ▼
DMS Processes Backlog
  │
  ▼
Steady State
```

Once the backlog is processed, the migration reaches a more stable state.

---

# 🔀 Application Cutover

When the target has caught up sufficiently, the application can be moved to the new database.

The lesson describes the process as:

```text
Source Database
      │
      ▼
DMS Migration
      │
      ▼
Target Catches Up
      │
      ▼
Stop Application
      │
      ▼
Complete Remaining Transactions
      │
      ▼
Point Application to Target
      │
      ▼
Start Application
```

This is the database migration cutover.

---

# 🧩 DMS Migration Types

The lesson introduces three migration modes:

| Migration Type | Purpose |
|---|---|
| Full Load | One-time migration of existing data |
| Full Load + CDC | Existing data plus ongoing changes |
| CDC Only | Replicate ongoing changes |

---

# 1. Full Load

A Full Load performs a:

```text
One-Time Migration
```

of existing data.

```text
Source
   │
   ▼
Existing Data
   │
   ▼
Target
```

This is appropriate when ongoing replication is not required.

---

# 2. Full Load + CDC

This combines:

```text
Existing Data

+

Ongoing Changes
```

Conceptually:

```text
Source
   │
   ├── Existing Data ──────► Full Load
   │
   └── New Changes ────────► CDC
                                │
                                ▼
                             Target
```

This allows the source to continue changing while the migration is taking place.

---

# 3. CDC Only

DMS can also be configured to capture:

```text
Ongoing Changes Only
```

This is useful when the initial data migration has already been completed or when long-running replication is required.

The lesson describes CDC as collecting changes from database logs using the database engine's native APIs.

---

# 🔄 Change Data Capture (CDC)

CDC stands for:

```text
Change Data Capture
```

It allows DMS to identify ongoing database changes such as:

```text
INSERT

UPDATE

DELETE
```

and replicate them to the target.

Mental model:

```text
Source Database
      │
      ▼
Database Logs
      │
      ▼
     CDC
      │
      ▼
    AWS DMS
      │
      ▼
Target Database
```

---

# 🏢 Long-Running Replication

CDC does not have to be used only during a short migration window.

The lesson also describes a scenario where the source continues running while DMS continuously replicates data to AWS.

```text
Active Source Database
        │
        ▼
      AWS DMS
        │
        ▼
AWS Target Database
```

This can support longer-running hybrid replication scenarios.

---

# 🔁 Homogeneous Migration

A:

```text
Homogeneous Migration
```

means the source and target use the same database engine.

Examples:

```text
MySQL
  │
  ▼
RDS MySQL
```

```text
Oracle
  │
  ▼
Oracle
```

```text
SQL Server
    │
    ▼
SQL Server
```

Because the database engines are the same, the schema does not need to be converted to a different database engine format.

---

# 🔀 Heterogeneous Migration

A:

```text
Heterogeneous Migration
```

means the source and target use different database engines.

Examples from the lesson include:

```text
Oracle
   │
   ▼
Aurora MySQL
```

or:

```text
SQL Server
    │
    ▼
RDS Oracle
```

Now there is an additional problem.

The source database schema may not be directly compatible with the target database engine.

Therefore:

```text
Source Schema
     │
     ▼
Schema Conversion
     │
     ▼
Target Schema
```

is required.

---

# 🛠️ AWS Schema Conversion Tool (SCT)

For heterogeneous migrations, the lesson introduces:

```text
Schema Conversion Tool
        │
        ▼
       SCT
```

SCT reads the source database schema and converts it into a format suitable for the target database.

---

# 🏗️ Heterogeneous Migration Architecture

The migration becomes a two-stage process.

## Stage 1: Convert the Schema

```text
Source Database
      │
      ▼
Source Schema
      │
      ▼
     SCT
      │
      ▼
Converted Schema
      │
      ▼
Target Database
```

## Stage 2: Migrate the Data

```text
Source Database
      │
      ▼
    AWS DMS
      │
      ▼
Target Database
```

Therefore:

```text
Heterogeneous Migration

Stage 1
SCT
Convert Schema

       +

Stage 2
DMS
Migrate Data
```

---

# 🆚 Homogeneous vs Heterogeneous Migration

| Feature | Homogeneous | Heterogeneous |
|---|---|---|
| Source/Target Engine | Same | Different |
| Example | MySQL → MySQL | Oracle → Aurora MySQL |
| Schema Conversion | Not required | Required |
| SCT | Not required | Used |
| DMS | Used for data migration | Used for data migration |

The most important distinction is:

```text
SAME ENGINE
     │
     ▼
Homogeneous
     │
     ▼
DMS


DIFFERENT ENGINE
     │
     ▼
Heterogeneous
     │
     ▼
SCT + DMS
```

---

# 🔎 DMS Fleet Advisor

The lesson also introduces:

```text
DMS Fleet Advisor
```

Fleet Advisor can help discover data stores in an on-premises environment that may be candidates for migration.

Conceptually:

```text
On-Premises Environment
          │
          ▼
    DMS Fleet Advisor
          │
          ▼
Discover Data Stores
          │
          ▼
Identify Migration Candidates
```

This can be useful during the migration assessment and discovery stage.

---

# 🪣 Amazon S3 as a DMS Target

AWS DMS can also migrate data from supported database sources into:

```text
Amazon S3
```

The architecture becomes:

```text
Source Database
      │
      ▼
Source Endpoint
      │
      ▼
DMS Replication Instance
      │
      ▼
S3 Target Endpoint
      │
      ▼
Amazon S3 Bucket
```

The source can include:

```text
On-Premises Database

Amazon RDS

EC2-Hosted Database
```

The replication instance extracts and transfers the data into Amazon S3.

---

# 📄 S3 Output Formats

The lesson introduces two output formats:

```text
CSV

Apache Parquet
```

By default:

```text
DMS → S3
    │
    ▼
   CSV
```

The data can alternatively be stored as:

```text
Apache Parquet
```

---

# 📊 Apache Parquet and Athena

The lesson highlights Apache Parquet as useful for:

```text
Amazon Athena
```

For example:

```text
Database
    │
    ▼
AWS DMS
    │
    ▼
Amazon S3
    │
    ▼
Parquet
    │
    ▼
Amazon Athena
```

This provides a useful pattern when database data needs to be moved into S3 for analytics.

---

# 📂 S3 Folder Organization

DMS can organize migrated data in S3 using a structure based on:

```text
Schema
   │
   ▼
Table
   │
   ▼
Files
```

Conceptually:

```text
S3 Bucket
│
├── Schema-A/
│   ├── Table-1/
│   │   ├── file1
│   │   └── file2
│   │
│   └── Table-2/
│
└── Schema-B/
```

This helps keep migrated datasets organized.

---

# ⚡ Parallel Loading

For large tables, the lesson also introduces:

```text
Parallel Loading
```

Large tables can be divided into segments or partitions and transferred using multiple threads.

Conceptually:

```text
Large Table
    │
    ├── Segment 1 ──►
    ├── Segment 2 ──► AWS DMS ──► S3
    ├── Segment 3 ──►
    └── Segment 4 ──►
```

The goal is to accelerate the transfer process.

---

# 🔄 CDC to Amazon S3

DMS can also capture ongoing database changes and send them to S3.

This means S3 can receive:

```text
Initial Full Load

+

INSERTS

UPDATES

DELETES
```

Conceptually:

```text
Source Database
      │
      ├── Full Load
      │
      └── CDC
             │
             ▼
          AWS DMS
             │
             ▼
          Amazon S3
```

This allows both one-time database exports and ongoing change capture.

---

# 🔐 IAM Requirements for S3

When using S3 as a target, the appropriate:

```text
IAM Role
```

and permissions need to be configured.

Conceptually:

```text
AWS DMS
   │
   │ IAM Permissions
   ▼
Amazon S3
```

DMS needs permission to perform the required operations against the target bucket.

---

# 🏗️ DMS to S3 Implementation Flow

The lesson summarizes the process as:

```text
1. Prepare S3 Bucket
        │
        ▼
2. Configure IAM Role
        │
        ▼
3. Launch Replication Instance
        │
        ▼
4. Configure Source Endpoint
        │
        ▼
5. Configure S3 Target Endpoint
        │
        ▼
6. Configure Migration Task
        │
        ▼
7. Configure Table Mappings
        │
        ▼
8. Run Migration
```

The lesson specifies that the replication instance should be in the same AWS Region as the target S3 bucket.

---

# 🔐 DMS Security

The lesson also introduces SSL protection for DMS endpoints.

Source and target endpoints can be configured using:

```text
SSL
```

and certificates can be assigned to endpoints through the DMS console or API.

Conceptually:

```text
Source
   │
   │ SSL
   ▼
AWS DMS
   │
   │ SSL
   ▼
Target
```

This helps protect data connections during migration.

---

# 🏗️ Complete Migration Architecture

A useful mental model for the overall service is:

```text
             SOURCE
     On-Premises / EC2
                │
                ▼
         Source Endpoint
                │
                ▼
       ┌─────────────────┐
       │     AWS DMS     │
       │                 │
       │   Replication   │
       │    Instance     │
       └────────┬────────┘
                │
                ▼
         Target Endpoint
                │
       ┌────────┼─────────┐
       ▼        ▼         ▼
      RDS    DynamoDB     S3
```

For different database engines:

```text
Source
  │
  ▼
SCT
  │
  ▼
Convert Schema
  │
  ▼
Target Schema

+

Source Data
  │
  ▼
DMS
  │
  ▼
Target Data
```

---

# 📊 Quick Reference

| Requirement | Service / Feature |
|---|---|
| Migrate database data | AWS DMS |
| One-time migration | Full Load |
| Existing data + ongoing changes | Full Load + CDC |
| Replicate only ongoing changes | CDC Only |
| Same source and target engine | Homogeneous Migration |
| Different source and target engines | Heterogeneous Migration |
| Convert database schema | SCT |
| Discover migration candidates | DMS Fleet Advisor |
| Database data to object storage | DMS → Amazon S3 |
| Default S3 format | CSV |
| Analytics-friendly format discussed | Apache Parquet |
| Protect endpoint connections | SSL |

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking DMS Converts Every Database Schema

DMS primarily handles the data migration and replication process.

For different source and target engines, the lesson introduces:

```text
Schema Conversion Tool
```

for schema conversion.

---

## Mistake 2: Using SCT for Every Migration

Remember:

```text
MySQL → MySQL
      │
      ▼
Homogeneous
      │
      ▼
No SCT Required
```

But:

```text
Oracle → Aurora MySQL
       │
       ▼
Heterogeneous
       │
       ▼
SCT + DMS
```

---

## Mistake 3: Thinking Full Load Automatically Means Continuous Replication

A simple Full Load is a one-time migration.

For ongoing changes, use:

```text
CDC
```

---

## Mistake 4: Forgetting the Replication Instance

The lesson's DMS architecture contains:

```text
Source
   │
   ▼
Replication Instance
   │
   ▼
Target
```

The replication instance performs the data replication.

---

## Mistake 5: Forgetting About Endpoints

DMS requires:

```text
Source Endpoint

+

Target Endpoint
```

to identify the source and destination.

---

## Mistake 6: Assuming DMS Only Migrates to Databases

Amazon S3 can also be used as a target.

```text
Database
    │
    ▼
AWS DMS
    │
    ▼
Amazon S3
```

---

# ❓ Interview Questions

### Q1. What is AWS DMS?

AWS Database Migration Service is a fully managed service used to migrate and replicate supported databases and data stores.

---

### Q2. What are the main components of a DMS migration?

```text
Source Database

Source Endpoint

Replication Instance

Migration Task

Target Endpoint

Target Database
```

---

### Q3. What does the replication instance do?

It performs the replication of data between the source and target.

---

### Q4. What are the three migration modes discussed?

```text
Full Load

Full Load + CDC

CDC Only
```

---

### Q5. What is Full Load?

A one-time migration of the existing data from the source to the target.

---

### Q6. What is CDC?

CDC stands for:

```text
Change Data Capture
```

It captures ongoing changes occurring in the source database and replicates them to the target.

---

### Q7. What is a homogeneous migration?

A migration where the source and target use the same database engine.

Example:

```text
MySQL → RDS MySQL
```

---

### Q8. What is a heterogeneous migration?

A migration where the source and target use different database engines.

Example:

```text
Oracle → Aurora MySQL
```

---

### Q9. When is SCT required?

The lesson uses SCT for heterogeneous migrations where the source and target database schemas need to be converted between different database engines.

---

### Q10. Is SCT required for a homogeneous migration?

No.

---

### Q11. What does SCT do?

It reads the source database schema and converts it into a schema suitable for the target database engine.

---

### Q12. What is DMS Fleet Advisor?

It helps discover on-premises data stores that may be candidates for migration.

---

### Q13. Can Amazon S3 be a DMS target?

Yes.

---

### Q14. What formats can DMS use when writing to S3 according to the lesson?

```text
CSV

Apache Parquet
```

---

### Q15. Which format is the default?

```text
CSV
```

---

### Q16. Why might Apache Parquet be useful?

The lesson highlights it as useful when the S3 data will be queried using services such as Amazon Athena.

---

### Q17. Can DMS continuously replicate database changes into S3?

Yes.

The lesson describes using CDC for inserts, updates, and deletes.

---

### Q18. Can DMS connections use SSL?

Yes.

The lesson describes configuring SSL and certificates for source and target endpoints.

---

### Q19. What happens to changes made while a Full Load is running?

The lesson describes those changes being cached after change capture begins for a table and applied after that table's full load completes.

---

### Q20. How would you migrate Oracle to Aurora MySQL?

Using the lesson's architecture:

```text
SCT
 │
 ▼
Convert Schema

+

DMS
 │
 ▼
Migrate Data
```

---

# 💡 Key Takeaways

- AWS DMS is a fully managed database migration service.
- It can migrate databases from on-premises or EC2-hosted environments to supported AWS targets.
- DMS architecture uses source and target endpoints.
- A replication instance performs the migration and replication work described in the lesson.
- Full Load migrates existing data.
- Full Load + CDC migrates existing data while continuing to capture changes.
- CDC can replicate ongoing changes.
- Homogeneous migrations use the same database engine.
- Heterogeneous migrations use different database engines.
- SCT is used for schema conversion in heterogeneous migrations.
- SCT is not required for homogeneous migrations.
- DMS Fleet Advisor helps discover migration candidates.
- Amazon S3 can be used as a DMS target.
- DMS can store migrated S3 data as CSV or Apache Parquet.
- CSV is the default format described in the lesson.
- Parallel loading can accelerate the transfer of large tables.
- IAM permissions are required when DMS writes to S3.
- SSL can protect DMS endpoint connections.

The easiest way to remember the service is:

```text
Need to MOVE DATABASE DATA?
          │
          ▼
        AWS DMS
```

Then determine the migration type:

```text
Same Database Engine?
        │
   ┌────┴────┐
   │         │
  YES        NO
   │         │
   ▼         ▼
Homogeneous  Heterogeneous
   │         │
   ▼         ▼
  DMS      SCT + DMS
```

And determine how much data to migrate:

```text
Existing Data Only
       │
       ▼
   FULL LOAD


Existing + New Changes
       │
       ▼
 FULL LOAD + CDC


Ongoing Changes Only
       │
       ▼
    CDC ONLY
```

---

# 📚 Related Topics

- AWS Database Migration Service (DMS)
- AWS Schema Conversion Tool (SCT)
- DMS Fleet Advisor
- Database Migration
- Homogeneous Migration
- Heterogeneous Migration
- Change Data Capture (CDC)
- Full Load
- Ongoing Replication
- Amazon RDS
- Amazon DynamoDB
- Amazon Redshift
- Amazon S3
- Apache Parquet
- Amazon Athena