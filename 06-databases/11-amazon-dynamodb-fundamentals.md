# ⚡ Amazon DynamoDB Fundamentals

> Amazon DynamoDB is AWS's fully managed, serverless NoSQL database service. It supports key-value and document data models, provides flexible schemas, and is designed to deliver single-digit millisecond performance at scale.

---

# 📖 Overview

So far, we have mainly looked at relational databases through services such as Amazon RDS and Amazon Aurora.

Relational databases organize information using structured:

```text
Tables
  │
  ├── Rows
  └── Columns
```

and normally require a predefined schema.

There is another category of database:

```text
NoSQL Database
      │
      ▼
Non-Relational Database
```

AWS provides a serverless NoSQL database service called:

```text
Amazon DynamoDB
```

DynamoDB is designed for modern applications that may need:

```text
High Performance

Massive Scale

Flexible Data Structures

Serverless Architecture
```

---

# ⚡ What Is Amazon DynamoDB?

Amazon DynamoDB is a:

```text
Serverless
+
Fully Managed
+
NoSQL Database
```

With DynamoDB, I don't need to provision a traditional database instance.

For example, I don't have to select:

```text
Database Instance Class

CPU

Memory

Attached Database Storage
```

AWS manages the underlying infrastructure.

The lesson describes DynamoDB as running on:

```text
SSD Storage
```

and being fully managed by AWS.

---

# 🆚 Relational vs NoSQL

A traditional relational database generally follows a predefined schema.

For example:

```text
Customer Table

┌────────────┬────────────┬────────────┐
│ CustomerID │ FirstName  │ LastName   │
├────────────┼────────────┼────────────┤
│ 101        │ John       │ Major      │
│ 102        │ Amy        │ Beer       │
└────────────┴────────────┴────────────┘
```

The columns are defined as part of the schema.

DynamoDB provides a more flexible approach.

Different items can contain different attributes.

---

# 🧠 Flexible Schema

Suppose I have three DynamoDB items:

```text
Item 101
│
├── PersonID
├── FirstName
├── LastName
└── Phone


Item 102
│
├── PersonID
├── FirstName
└── LastName


Item 103
│
├── PersonID
├── FirstName
├── LastName
├── FavoriteColor
└── Address
```

The items do not all need to contain exactly the same attributes.

This provides:

```text
Flexible Schema
```

instead of requiring every item to conform to the same predefined structure.

---

# 🎯 DynamoDB Performance

The lesson describes DynamoDB as capable of providing:

```text
Single-Digit Millisecond Performance
```

across a wide range of workloads.

It can support applications ranging from a small number of users to workloads involving:

```text
Tens of Millions of Users
```

This makes DynamoDB suitable for modern applications requiring high scalability.

---

# 🗃️ DynamoDB Data Models

DynamoDB supports:

```text
Key-Value Data Model

Document Data Model
```

This allows applications to store:

```text
Semi-Structured Data
```

without requiring the rigid table structure normally associated with relational databases.

---

# 🆚 RDS vs DynamoDB

| Area           | Amazon RDS                                  | Amazon DynamoDB                    |
| -------------- | ------------------------------------------- | ---------------------------------- |
| Database Type  | Relational                                  | NoSQL                              |
| Architecture   | Database instances                          | Serverless                         |
| Schema         | Structured/predefined                       | Flexible                           |
| Relationships  | Supports relational relationships and joins | No complex relational joins        |
| Data Model     | Relational tables                           | Key-value and document             |
| Infrastructure | Instance configuration required             | AWS managed                        |
| Scaling        | Depends on RDS configuration                | Designed for large-scale workloads |

An easy way to remember this is:

```text
RDS
 │
 ▼
Relational
 │
 ▼
Structured Schema
 │
 ▼
Complex Relationships


DynamoDB
 │
 ▼
NoSQL
 │
 ▼
Flexible Schema
 │
 ▼
Key-Value / Document
```

---

# 🏗️ DynamoDB Data Structure

The main DynamoDB concepts introduced in this lesson are:

```text
Table
  │
  ├── Item
  │     │
  │     └── Attributes
  │
  ├── Item
  │     │
  │     └── Attributes
  │
  └── Item
        │
        └── Attributes
```

The terminology is important.

---

# 📋 Table

A:

```text
Table
```

contains the application data.

Unlike a traditional database service, I don't create a database instance first and then create tables inside that instance.

The lesson describes:

```text
DynamoDB
   │
   ├── Table A
   ├── Table B
   └── Table C
```

The tables are created directly within DynamoDB.

---

# 📦 Items

Inside a DynamoDB table are:

```text
Items
```

An item is similar to a:

```text
Row

or

Record
```

in a relational database.

For example:

```text
People Table
│
├── Item 101
├── Item 102
└── Item 103
```

---

# 🏷️ Attributes

Each item contains:

```text
Attributes
```

Attributes describe the item.

For example:

```text
Item
│
├── PersonID = 101
├── FirstName = John
├── LastName = Major
└── Phone = 123456789
```

Attributes are somewhat comparable to fields or columns in a relational database.

---

# 🧩 Different Items Can Have Different Attributes

One of DynamoDB's important characteristics is that items don't need identical attributes.

For example:

```text
Item 101
├── PersonID
├── FirstName
├── LastName
└── Phone


Item 103
├── PersonID
├── FirstName
├── LastName
├── FavoriteColor
└── Address
```

`FavoriteColor` and `Address` may exist only for Item 103.

The other items don't need those attributes.

---

# 🪆 Nested Attributes

Attributes can also contain nested information.

For example:

```text
Person
│
├── PersonID
├── FirstName
├── LastName
└── Address
      │
      ├── Street
      ├── Town
      ├── City
      └── Postcode
```

The lesson describes nested attributes as supporting up to:

```text
32 Levels
```

of nesting.

---

# 🔑 Primary Key

Every DynamoDB table must have a:

```text
Primary Key
```

The primary key uniquely identifies an item in the table.

For example:

```text
PersonID
   │
   ├── 101 → John
   ├── 102 → David
   └── 103 → Amy
```

Each key uniquely identifies its corresponding item.

---

# 🧭 Partition Key

The primary key can also act as the:

```text
Partition Key
```

DynamoDB uses the partition key value as input to an internal:

```text
Hash Function
```

Conceptually:

```text
Partition Key
      │
      ▼
 Hash Function
      │
      ▼
Determine Partition
      │
      ▼
Store Item
```

AWS manages the physical storage and partition placement transparently.

---

# 📦 Maximum Item Size

The lesson states that an individual DynamoDB item can be up to:

```text
400 KB
```

in size.

---

# 🔑 Simple Primary Key

A table can use only a partition key as its primary key.

For example:

```text
PersonID
   │
   ├── 101
   ├── 102
   └── 103
```

Each partition key value must uniquely identify the item.

---

# 🔑 Partition Key + Sort Key

DynamoDB can also use a combination of:

```text
Partition Key

+

Sort Key
```

Together, they uniquely identify the item.

Conceptually:

```text
Partition Key      Sort Key
     │                 │
     └────────┬────────┘
              ▼
       Unique Item
```

This allows multiple items to have the same partition key as long as their sort keys are different.

---

# 🧩 Composite Key Example

Suppose the partition key is:

```text
CustomerID
```

and the sort key is:

```text
OrderID
```

I could have:

```text
CustomerID    OrderID

101           ORDER-1
101           ORDER-2
101           ORDER-3

102           ORDER-1
102           ORDER-2
```

`CustomerID = 101` appears multiple times.

However:

```text
101 + ORDER-1
101 + ORDER-2
101 + ORDER-3
```

are unique combinations.

---

# 📊 Sort Key Behavior

Items with the same partition key are stored together and ordered according to the sort key.

Conceptually:

```text
Partition Key = 101
│
├── Sort Key = 001
├── Sort Key = 002
└── Sort Key = 003
```

This allows related items to share the same partition key while remaining uniquely identifiable.

---

# 🏷️ DynamoDB Table Classes

The lesson introduces two DynamoDB table classes:

```text
DynamoDB Standard

DynamoDB Standard-Infrequent Access
```

These help optimize costs for different storage patterns.

---

# 📘 DynamoDB Standard

```text
DynamoDB Standard
```

is the default table class.

The lesson recommends it for:

```text
The Vast Majority
of Workloads
```

---

# 📦 DynamoDB Standard-Infrequent Access

The:

```text
DynamoDB Standard-Infrequent Access
```

table class is designed for tables where:

```text
Storage
   │
   ▼
Dominant Cost
```

and the data is accessed less frequently.

Examples from the lesson include:

```text
Application Logs

Social Media Posts

E-Commerce Order History

Past Gaming Achievements
```

---

# 🆚 DynamoDB Table Classes

| Table Class                | Suitable For                                                  |
| -------------------------- | ------------------------------------------------------------- |
| DynamoDB Standard          | Most workloads                                                |
| Standard-Infrequent Access | Infrequently accessed data where storage is the dominant cost |

---

# ⚙️ DynamoDB Capacity Modes

DynamoDB provides two capacity options:

```text
On-Demand Capacity

Provisioned Capacity
```

These determine how read and write capacity is managed and billed.

---

# 🚀 On-Demand Capacity

The lesson describes:

```text
On-Demand
```

as the default capacity mode.

It uses a:

```text
Pay-Per-Request
```

pricing model.

The reads and writes performed by applications are measured using:

```text
Read Request Units

Write Request Units
```

Conceptually:

```text
Application Requests
       │
       ▼
     DynamoDB
       │
       ▼
Actual Requests
       │
       ▼
Pay for Requests
```

---

# 🎯 When to Use On-Demand

The lesson identifies on-demand capacity as suitable for:

```text
Unpredictable Workloads

Zero Administration

Rapidly Changing Demand
```

It can start small and scale to very large request volumes.

---

# 📊 Provisioned Capacity

With:

```text
Provisioned Capacity
```

I specify how much read and write throughput the table should support.

Conceptually:

```text
DynamoDB Table
      │
      ├── Read Capacity
      │
      └── Write Capacity
```

Provisioned capacity is managed:

```text
Per Table
```

---

# 🎯 When to Use Provisioned Capacity

The lesson describes provisioned capacity as suitable for:

```text
Predictable

and

Stable Workloads
```

where I can determine the expected read and write requirements.

---

# 🆚 On-Demand vs Provisioned Capacity

| Area                | On-Demand             | Provisioned        |
| ------------------- | --------------------- | ------------------ |
| Pricing             | Pay per request       | Provision capacity |
| Capacity Planning   | Minimal               | Required           |
| Workload            | Unpredictable         | Predictable/stable |
| Read/Write Capacity | Automatically handled | Defined per table  |
| Administration      | Lower                 | More control       |

Easy memory aid:

```text
UNPREDICTABLE
      │
      ▼
   ON-DEMAND


PREDICTABLE
      │
      ▼
  PROVISIONED
```

---

# 📖 Read Capacity Units

With provisioned capacity, read throughput is measured using:

```text
Read Capacity Units
       │
       ▼
      RCU
```

According to the lesson:

### One RCU provides:

```text
1 Strongly Consistent Read
per second
for an item up to 4 KB
```

or:

```text
2 Eventually Consistent Reads
per second
for items up to 4 KB
```

---

# 🧮 RCU Example

Suppose an application needs:

```text
50 Items per Second

Each Item = 3 KB

Strongly Consistent Reads
```

One strongly consistent RCU supports an item up to:

```text
4 KB
```

Calculate the capacity needed per item:

```text
3 KB ÷ 4 KB
=
0.75
```

Round up:

```text
1 RCU per Item
```

For 50 items:

```text
50 × 1 RCU
=
50 RCUs
```

Therefore:

```text
Required Read Capacity
=
50 RCUs
```

---

# ✍️ Write Capacity Units

Write throughput is measured using:

```text
Write Capacity Units
       │
       ▼
      WCU
```

According to the lesson:

```text
1 WCU
=
1 Write per Second
for an Item up to 1 KB
```

---

# 🧮 WCU Example

Suppose the application writes:

```text
20 Items per Second

Each Item = 8 KB
```

For each item:

```text
8 KB ÷ 1 KB
=
8 WCUs
```

For 20 items:

```text
20 × 8
=
160 WCUs
```

Therefore:

```text
Required Write Capacity
=
160 WCUs
```

---

# ⚡ DynamoDB Burst Capacity

DynamoDB also provides:

```text
Burst Capacity
```

When provisioned throughput is not fully consumed, DynamoDB can retain some unused capacity.

Conceptually:

```text
Provisioned Capacity
       │
       ├── Used Capacity
       │
       └── Unused Capacity
                │
                ▼
          Burst Capacity
```

This unused capacity can help handle temporary workload spikes.

---

# 📈 Burst Example

Normally:

```text
Provisioned Throughput
        │
        ▼
Regular Workload
```

Then a temporary spike occurs:

```text
Unexpected Traffic Spike
          │
          ▼
Additional Requests
          │
          ▼
Burst Capacity
          │
          ▼
Requests Can Succeed
```

Without the additional capacity, some requests might otherwise be throttled.

---

# ⏱️ Burst Capacity Retention

The lesson states that DynamoDB can retain up to:

```text
5 Minutes

or

300 Seconds
```

of unused read and write capacity.

This retained capacity can be consumed during occasional bursts.

The lesson also notes that DynamoDB may use some of this capacity for internal background maintenance activities.

---

# 📖 Read Consistency

Another important DynamoDB concept is:

```text
Read Consistency
```

The lesson introduces three types:

```text
Eventually Consistent Reads

Strongly Consistent Reads

Transactional Reads
```

---

# 🔄 Eventually Consistent Reads

Eventually consistent reads are described as the:

```text
Default
```

read consistency option.

When data is written, it is replicated across nodes in multiple Availability Zones.

Conceptually:

```text
Write
 │
 ▼
Leader Node
 │
 ├────────► Replica
 │
 └────────► Replica
```

Replication takes some amount of time.

Therefore, immediately after a write, a read might not return the latest version of the data.

---

# 📦 Eventual Consistency Example

Suppose:

```text
10:00:00
Product Price = $100

10:00:01
Update Price = $90
```

An immediate eventually consistent read could temporarily return the older value before replication completes.

The lesson identifies workloads such as:

```text
Product Catalogs
```

as examples where immediately seeing the latest write may not always be necessary.

---

# 💰 Eventually Consistent Read Capacity

According to the lesson:

```text
1 RCU
=
2 Eventually Consistent Reads
per second
for items up to 4 KB
```

Another way of thinking about it is:

```text
1 Eventually Consistent Read
=
0.5 RCU
```

for an item up to 4 KB.

---

# 🔒 Strongly Consistent Reads

A:

```text
Strongly Consistent Read
```

returns the most up-to-date data reflecting successful writes completed before the read.

Conceptually:

```text
Write Data
    │
    ▼
Write Successful
    │
    ▼
Strong Read
    │
    ▼
Latest Data
```

---

# 🎯 Strong Consistency Use Cases

The lesson gives examples where accuracy may be critical:

```text
Financial Transactions

Session State Data

Medical Records
```

In these scenarios, applications may need the latest successful write.

---

# 💰 Strong Read Capacity

According to the lesson:

```text
1 RCU
=
1 Strongly Consistent Read
per second
for an item up to 4 KB
```

Compared with eventual consistency:

```text
1 RCU

Strong:
1 Read

Eventual:
2 Reads
```

Therefore, strongly consistent reads consume more read capacity.

---

# 🔄 Transactional Reads

The third type introduced is:

```text
Transactional Reads
```

These are used as part of DynamoDB transactions.

The lesson references:

```text
TransactGetItems API
```

for this type of operation.

---

# 🧩 Transactional Behavior

Transactional reads provide:

```text
All-or-Nothing Behavior
```

across multiple items and potentially multiple tables.

Conceptually:

```text
Transaction
│
├── Read Item A
├── Read Item B
└── Read Item C
        │
        ▼
All Successful?
   │          │
  YES         NO
   │          │
   ▼          ▼
Return      Return None
Items
```

If one part of the transaction fails, the transaction does not return only a partial result.

---

# 🎯 Transactional Read Use Cases

Transactional reads are useful where consistency is required across multiple related items.

For example:

```text
User Information

+

Pricing Information

+

Related Application Data
```

that needs to be retrieved together consistently.

---

# 💰 Transactional Read Capacity

The lesson states:

```text
1 Transactional Read
for an item up to 4 KB
=
2 RCUs
```

Therefore:

```text
Eventual Read
     │
     ▼
Lowest Capacity Consumption


Strong Read
     │
     ▼
Higher Capacity Consumption


Transactional Read
     │
     ▼
Highest of These Three
```

---

# 📊 Read Consistency Comparison

| Read Type             | Behavior                                         | Capacity for Item up to 4 KB |
| --------------------- | ------------------------------------------------ | ---------------------------: |
| Eventually Consistent | May temporarily return older data                |                      0.5 RCU |
| Strongly Consistent   | Returns latest successful write                  |                        1 RCU |
| Transactional         | Used for all-or-nothing transactional operations |                       2 RCUs |

---

# 🏗️ DynamoDB High-Level Architecture

Putting the concepts together:

```text
                       Application
                            │
                            ▼
                        DynamoDB
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
           Table A       Table B       Table C
              │
              ▼
            Items
              │
              ▼
          Attributes
              │
              ▼
        Primary / Partition Key
              │
              ▼
          Hash Function
              │
              ▼
           Partition
```

AWS manages the underlying infrastructure and storage.

---

# 🧠 DynamoDB Mental Model

The easiest way to think about DynamoDB is:

```text
DynamoDB
   │
   ├── Serverless
   │
   ├── NoSQL
   │
   ├── Key-Value / Document
   │
   ├── Flexible Schema
   │
   ├── Fully Managed
   │
   ├── Single-Digit Millisecond Performance
   │
   └── Scalable
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking DynamoDB Is a Relational Database

DynamoDB is:

```text
NoSQL
```

and does not provide the same complex relational join functionality as RDS.

---

## Mistake 2: Looking for a DynamoDB Database Instance

With DynamoDB:

```text
No Traditional DB Instance
```

I create tables directly in the DynamoDB service.

---

## Mistake 3: Thinking Every Item Must Have the Same Attributes

DynamoDB supports a flexible schema.

```text
Item A
├── ID
├── Name
└── Phone


Item B
├── ID
├── Name
├── Address
└── FavoriteColor
```

Both structures can exist within the same table.

---

## Mistake 4: Forgetting the Primary Key

Every DynamoDB table requires a primary key.

It uniquely identifies the table's items.

---

## Mistake 5: Thinking Duplicate Partition Keys Are Always Invalid

With only a partition key:

```text
Partition Key
=
Unique
```

But with a partition key and sort key:

```text
Partition Key + Sort Key
=
Unique Combination
```

Multiple items can therefore share a partition key if their sort keys differ.

---

## Mistake 6: Confusing On-Demand and Provisioned Capacity

```text
Unpredictable
     │
     ▼
On-Demand


Predictable
     │
     ▼
Provisioned
```

---

## Mistake 7: Forgetting to Round Capacity Calculations

Capacity calculations must account for the supported item-size unit.

For example:

```text
3 KB Strong Read

3 ÷ 4
=
0.75

Round Up
=
1 RCU
```

---

## Mistake 8: Assuming Eventual Consistency Always Returns the Latest Data

Eventually consistent reads may temporarily return an older version after a recent write.

---

# ❓ Interview Questions

### Q1. What is Amazon DynamoDB?

Amazon DynamoDB is AWS's fully managed, serverless NoSQL database service supporting key-value and document data models.

---

### Q2. Do I need to provision a database instance for DynamoDB?

No.

DynamoDB uses a serverless architecture and AWS manages the underlying infrastructure.

---

### Q3. What performance does the lesson associate with DynamoDB?

```text
Single-Digit Millisecond Performance
```

---

### Q4. What data models does DynamoDB support?

```text
Key-Value

Document
```

---

### Q5. What are the main DynamoDB data structure terms?

```text
Table

Item

Attribute
```

---

### Q6. What is an item?

An item is comparable to a row or record in a relational database.

---

### Q7. What is an attribute?

An attribute describes information about an item.

---

### Q8. Do all items need the same attributes?

No.

DynamoDB provides a flexible schema where different items can contain different sets of attributes.

---

### Q9. How deeply can attributes be nested according to the lesson?

```text
Up to 32 Levels
```

---

### Q10. What is the maximum DynamoDB item size?

```text
400 KB
```

---

### Q11. Why does DynamoDB require a primary key?

The primary key uniquely identifies each item in the table.

---

### Q12. What is the partition key used for?

DynamoDB uses the partition key value as input to an internal hash function to determine where an item is stored.

---

### Q13. What is a sort key?

A sort key can be combined with a partition key to create a composite key.

This allows multiple items to share the same partition key while remaining uniquely identifiable.

---

### Q14. What are the DynamoDB table classes discussed?

```text
DynamoDB Standard

DynamoDB Standard-Infrequent Access
```

---

### Q15. When is Standard-Infrequent Access useful?

For infrequently accessed tables where storage is the dominant cost.

Examples in the lesson include logs, social media posts, order history, and historical gaming achievements.

---

### Q16. What are DynamoDB's two capacity modes?

```text
On-Demand

Provisioned
```

---

### Q17. When should I consider On-Demand capacity?

For unpredictable workloads or when I want minimal capacity administration.

---

### Q18. When should I consider Provisioned capacity?

For predictable and stable workloads where expected read and write throughput can be defined.

---

### Q19. What does one RCU provide for strongly consistent reads?

```text
1 Strongly Consistent Read
per second
for an item up to 4 KB
```

---

### Q20. What does one RCU provide for eventually consistent reads?

```text
2 Eventually Consistent Reads
per second
for items up to 4 KB
```

---

### Q21. What does one WCU provide?

```text
1 Write per Second
for an item up to 1 KB
```

---

### Q22. What is DynamoDB burst capacity?

DynamoDB can retain unused read and write capacity and use it to handle temporary bursts of activity.

---

### Q23. How much unused capacity does the lesson say DynamoDB can retain?

```text
Up to 5 Minutes

or

300 Seconds
```

---

### Q24. What is an eventually consistent read?

It may not immediately return the latest write because data replication may still be occurring.

---

### Q25. What is a strongly consistent read?

It returns the most up-to-date data reflecting successful writes completed before the read.

---

### Q26. What is a transactional read?

A transactional read is part of a DynamoDB transaction and provides all-or-nothing behavior across related items.

---

# 💡 Key Takeaways

* DynamoDB is AWS's serverless NoSQL database service.
* AWS manages the underlying database infrastructure.
* DynamoDB uses SSD storage.
* It supports key-value and document data models.
* It provides a flexible schema.
* Different items can contain different attributes.
* Nested attributes can be up to 32 levels deep.
* DynamoDB is designed for single-digit millisecond performance.
* Every DynamoDB table requires a primary key.
* A partition key determines item placement using an internal hash function.
* Partition key + sort key can form a composite primary key.
* Maximum item size is 400 KB.
* DynamoDB Standard is the default table class.
* Standard-Infrequent Access is designed for infrequently accessed data where storage is the dominant cost.
* DynamoDB supports On-Demand and Provisioned capacity modes.
* On-Demand is useful for unpredictable workloads.
* Provisioned capacity is useful for predictable workloads.
* Read throughput is measured using RCUs.
* Write throughput is measured using WCUs.
* DynamoDB can retain unused throughput as burst capacity.
* Eventually consistent reads are the default option described in the lesson.
* Strongly consistent reads return the latest successful write.
* Transactional reads provide all-or-nothing behavior for transaction operations.

The simplest mental model is:

```text
                    AMAZON DYNAMODB
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
       NoSQL          Serverless       Flexible
                                         Schema
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                        Table
                          │
                          ▼
                        Items
                          │
                          ▼
                      Attributes
                          │
                          ▼
                Primary / Partition Key
```

For capacity:

```text
             DYNAMODB CAPACITY
                    │
             ┌──────┴──────┐
             │             │
             ▼             ▼
         On-Demand     Provisioned
             │             │
             ▼             ▼
       Unpredictable    Predictable
         Workload        Workload
```

And for consistency:

```text
EVENTUAL
   │
   ▼
May Temporarily Return
Older Data
   │
   ▼
Lower Read Capacity Cost


STRONG
   │
   ▼
Latest Successful Write
   │
   ▼
Higher Read Capacity Cost


TRANSACTIONAL
   │
   ▼
All-or-Nothing
Multi-Item Operations
```

---

# 📚 Related Topics

* NoSQL Databases
* Amazon DynamoDB
* DynamoDB Tables
* Items and Attributes
* Partition Keys
* Sort Keys
* Composite Keys
* DynamoDB Standard
* DynamoDB Standard-Infrequent Access
* On-Demand Capacity
* Provisioned Capacity
* Read Capacity Units (RCUs)
* Write Capacity Units (WCUs)
* Burst Capacity
* Eventually Consistent Reads
* Strongly Consistent Reads
* DynamoDB Transactions
