# 🗄️ Relational Databases and Amazon RDS Fundamentals

> Relational databases organize related data into structured tables. Relationships between those tables allow applications to store, retrieve, and analyze data efficiently.

---

# 📖 Overview

Almost every application needs to work with data.

For example:

```text
E-Commerce Application
        │
        ├── Customers
        ├── Orders
        ├── Products
        ├── Payments
        └── Addresses
```

As an application grows, storing everything in one large file or spreadsheet becomes difficult to manage.

A relational database solves this by organizing data into:

```text
Multiple Tables
      │
      ▼
Relationships
      │
      ▼
Structured Data
```

The main concepts I need to understand are:

```text
Relational Database
│
├── Tables
├── Rows
├── Columns
├── Primary Keys
├── Foreign Keys
├── Relationships
├── Joins
├── Schema
├── SQL
├── CRUD
└── ACID
```

Once I understand these concepts, Amazon RDS becomes much easier to understand.

---

# 🎯 Why Do We Need Databases?

Imagine an e-commerce application.

I need to store:

```text
Customer Information

Order Information

Product Information

Order Dates

Order Amounts
```

I could initially put everything into one spreadsheet:

```text
Customer ID
First Name
Last Name
Address
Order ID
Order Date
Order Amount
Product
Quantity
...
```

But imagine:

```text
1,000 Customers

10 Orders per Customer

Multiple Products per Order
```

The spreadsheet quickly becomes:

```text
Large

Repetitive

Difficult to Search

Difficult to Maintain

Difficult to Update
```

For example, the same customer information may appear repeatedly:

```text
Edward Smith | Address | Order-001
Edward Smith | Address | Order-002
Edward Smith | Address | Order-003
```

Instead, we can separate the information.

---

# 🗃️ Relational Database

A relational database organizes information into:

```text
Tables
```

and those tables can be related to each other.

Instead of:

```text
One Huge Table
```

we could have:

```text
Customers Table

Orders Table

Products Table
```

and create relationships between them.

For example:

```text
Customers
    │
    │ Customer_ID
    ▼
Orders
```

Now customer information does not need to be repeated for every order.

---

# 📊 Database Tables

A relational database table consists of:

```text
Rows
   +
Columns
```

Example:

| Customer_ID | First_Name | Last_Name | Town    |
| ----------- | ---------- | --------- | ------- |
| C001        | John       | Major     | London  |
| C002        | Edward     | Smith     | Bristol |
| C003        | Marcus     | Jones     | Leeds   |

---

# 📌 Rows

A row represents:

```text
One Record
```

For example:

```text
C002 | Edward | Smith | Bristol
```

is one customer record.

So:

```text
Row
 =
Record
```

---

# 📌 Columns

Columns represent:

```text
Attributes
```

of the record.

For example:

```text
Customer
│
├── Customer_ID
├── First_Name
├── Last_Name
└── Town
```

So:

```text
Column
   =
Attribute / Field
```

---

# 🔑 Primary Key

Every record needs a way to be uniquely identified.

This is where a:

```text
Primary Key
```

is used.

Example:

| Customer_ID | First_Name | Last_Name |
| ----------- | ---------- | --------- |
| C001        | John       | Major     |
| C002        | Edward     | Smith     |
| C003        | Marcus     | Jones     |

Here:

```text
Customer_ID
```

is the primary key.

The important property is:

```text
Primary Key
     │
     ▼
Uniquely Identifies
One Record
```

Therefore:

```text
C001
C002
C003
```

must identify different customer records.

---

# 🧠 Why Do We Need a Primary Key?

Imagine two customers have the same name:

```text
John Smith

John Smith
```

The name cannot reliably identify which customer I mean.

Instead:

```text
C001 → John Smith

C847 → John Smith
```

Now the application can uniquely identify each customer.

---

# 🛒 Separate Orders Table

Instead of storing orders directly with customer details, I can create another table.

Example:

| Order_ID | Customer_ID | Order_Date | Amount |
| -------- | ----------- | ---------- | -----: |
| O1001    | C002        | 16-May     |    £98 |
| O1002    | C002        | 20-May     |    £45 |
| O1003    | C003        | 21-May     |   £120 |

Here:

```text
Order_ID
```

is the primary key of the Orders table.

But:

```text
Customer_ID
```

connects the order back to the customer.

This is a:

```text
Foreign Key
```

---

# 🔗 Foreign Key

A Foreign Key creates a relationship with another table.

Example:

```text
CUSTOMERS TABLE

Customer_ID
    C001
    C002
    C003
      │
      │
      │ Relationship
      ▼

ORDERS TABLE

Order_ID    Customer_ID
O1001       C002
O1002       C002
O1003       C003
```

In the Customers table:

```text
Customer_ID
     =
Primary Key
```

In the Orders table:

```text
Customer_ID
     =
Foreign Key
```

---

# 🧠 Primary Key vs Foreign Key

The easiest way to remember:

```text
PRIMARY KEY
     │
     ▼
Identifies the record
inside THIS table


FOREIGN KEY
     │
     ▼
References a record
in ANOTHER table
```

For example:

```text
Customers
-------------------
Customer_ID (PK)
Name


Orders
-------------------
Order_ID (PK)
Customer_ID (FK)
Amount
```

---

# 🔁 One Customer Can Have Multiple Orders

One important concept from this example is:

```text
One Customer
     │
     ├── Order 1
     ├── Order 2
     └── Order 3
```

Therefore:

```text
Customer_ID = C002
```

can appear multiple times in the Orders table.

That is okay because it is a:

```text
Foreign Key
```

But:

```text
Order_ID
```

must uniquely identify each order because it is the primary key of that table.

---

# 🔗 Relationships

Relationships are fundamental to relational databases.

For example:

```text
CUSTOMER
    │
    │ places
    ▼
ORDER
```

or:

```text
CUSTOMER
    │
    ▼
ORDERS
    │
    ▼
ORDER ITEMS
    │
    ▼
PRODUCTS
```

This allows us to separate data logically while still connecting related information.

---

# 🔀 Joins

Sometimes I need information from multiple tables.

For example:

> Show me the customer name and all orders placed by that customer.

Customer information exists in:

```text
Customers Table
```

while order information exists in:

```text
Orders Table
```

The relationship allows the database to combine the information.

Conceptually:

```text
Customers
    │
    │ Customer_ID
    ▼
   JOIN
    ▲
    │ Customer_ID
    │
Orders
```

Result:

```text
Edward Smith
   │
   ├── Order-001
   ├── Order-002
   └── Order-003
```

---

# 🏗️ Database Schema

Before storing data in a relational database, we normally define how the database is structured.

This structure is called the:

```text
Database Schema
```

The schema describes things such as:

```text
Tables

Columns

Data Structure

Primary Keys

Relationships
```

For example:

```text
DATABASE
│
├── Customers
│    ├── Customer_ID
│    ├── First_Name
│    ├── Last_Name
│    └── Address
│
└── Orders
     ├── Order_ID
     ├── Customer_ID
     ├── Order_Date
     └── Amount
```

This is part of the database schema.

---

# 🎯 Why Is Schema Design Important?

If the database structure is poorly designed:

```text
Poor Schema
    │
    ▼
Duplicate Data
    │
    ▼
Difficult Queries
    │
    ▼
Difficult Maintenance
```

Good planning helps organize the data before large amounts of information are inserted.

Therefore:

```text
Plan
  │
  ▼
Define Schema
  │
  ▼
Create Tables
  │
  ▼
Insert Data
```

---

# 💻 SQL

Relational databases commonly use:

```text
SQL
```

which stands for:

```text
Structured Query Language
```

SQL is used to:

```text
Query Data

Insert Data

Update Data

Delete Data

Manage Data
```

---

# 🔎 SELECT

To retrieve information:

```sql
SELECT * FROM customers;
```

Conceptually:

```text
SELECT
   =
Read Data
```

---

# ➕ INSERT

To add information:

```sql
INSERT INTO customers
VALUES ('C001', 'John', 'Major');
```

Conceptually:

```text
INSERT
   =
Add Data
```

---

# ✏️ UPDATE

To change existing information:

```sql
UPDATE customers
SET town = 'Toronto'
WHERE customer_id = 'C001';
```

Conceptually:

```text
UPDATE
   =
Modify Data
```

---

# 🗑️ DELETE

To remove information:

```sql
DELETE FROM customers
WHERE customer_id = 'C001';
```

Conceptually:

```text
DELETE
   =
Remove Data
```

---

# 🔄 CRUD Operations

A common database concept is:

```text
CRUD
```

CRUD represents four fundamental operations:

```text
C → Create

R → Read

U → Update

D → Delete
```

---

# 🧠 CRUD and SQL

A useful mapping is:

| CRUD   | Purpose       | SQL Example |
| ------ | ------------- | ----------- |
| Create | Add data      | `INSERT`    |
| Read   | Retrieve data | `SELECT`    |
| Update | Modify data   | `UPDATE`    |
| Delete | Remove data   | `DELETE`    |

Easy memory aid:

```text
CRUD

Create
Read
Update
Delete
```

These are basic operations that applications perform against databases.

---

# 💳 Database Transactions

Another important relational database concept is:

```text
Transaction
```

A transaction represents a unit of work performed against the database.

For example:

```text
Transfer £100

Account A
    │
    │ -£100
    ▼
Database
    │
    │ +£100
    ▼
Account B
```

We do not want:

```text
Account A loses £100
       │
       ▼
Something Fails
       │
       ▼
Account B receives nothing
```

The database needs mechanisms to protect the integrity of transactions.

This leads to:

```text
ACID
```

---

# 🧪 ACID Properties

ACID stands for:

```text
A → Atomicity

C → Consistency

I → Isolation

D → Durability
```

These properties help ensure reliable transaction processing.

---

# 1. Atomicity

Atomicity means:

> A transaction completes completely or does not complete at all.

Think:

```text
ALL
or
NOTHING
```

Example:

```text
Transfer £100
     │
     ├── Deduct £100 from Account A
     │
     └── Add £100 to Account B
```

Both operations should succeed.

If something fails:

```text
Transaction Failure
        │
        ▼
Rollback
        │
        ▼
Original State
```

We should not end up with half of the transaction completed.

Easy memory aid:

```text
Atomicity
    =
All or Nothing
```

---

# 2. Consistency

Consistency means that transactions follow the database rules and move the database from one valid state to another.

Example:

```text
Before Transfer

Account A = £500
Account B = £200

Total = £700
```

Transfer:

```text
£100
A ─────────► B
```

After:

```text
Account A = £400
Account B = £300

Total = £700
```

The data remains valid and consistent.

Easy memory aid:

```text
Consistency
     =
Data Remains Valid
```

---

# 3. Isolation

Databases can have many users and applications performing transactions simultaneously.

For example:

```text
User A ──► Transaction A

User B ──► Transaction B

User C ──► Transaction C
```

Isolation ensures transactions do not incorrectly interfere with each other.

Conceptually:

```text
Transaction A
     │
     │ Independent
     │
Transaction B
```

This helps prevent issues such as one transaction seeing incomplete changes from another transaction.

Easy memory aid:

```text
Isolation
    =
Transactions Don't
Incorrectly Interfere
```

---

# 4. Durability

Durability means:

> Once a transaction has been successfully committed, its result is permanently recorded.

Conceptually:

```text
Transaction
     │
     ▼
COMMIT
     │
     ▼
Recorded
     │
     ▼
System Failure
     │
     ▼
Committed Data Remains
```

Easy memory aid:

```text
Durability
    =
Committed Means Persistent
```

---

# 🧠 ACID Memory Trick

```text
A
Atomicity
=
All or Nothing


C
Consistency
=
Valid State


I
Isolation
=
Independent Transactions


D
Durability
=
Committed Data Persists
```

---

# ☁️ Running Relational Databases on AWS

Now that I understand relational database fundamentals, I need to understand how I can run one on AWS.

The lesson introduces two approaches:

```text
Relational Database on AWS
          │
          ├── Database on EC2
          │
          └── Amazon RDS
```

These approaches give me different levels of responsibility.

---

# 1. Database on Amazon EC2

The first option is:

```text
Amazon EC2
     │
     ▼
Operating System
     │
     ▼
Install Database
```

For example:

```text
EC2
 │
 └── Linux
      │
      └── MySQL
```

or:

```text
EC2
 │
 └── Windows
      │
      └── Microsoft SQL Server
```

I can install and operate the database software myself.

---

# 🛠️ My Responsibilities with a Database on EC2

When I install a database on EC2, I am responsible for much of its administration.

Examples from the lesson include:

```text
Database Installation

Database Configuration

Database Backups

Database Patching

Operating System Patching

Scaling

Availability

Security Software

Database Licensing

Performance Optimization
```

Conceptually:

```text
AWS
 │
 ▼
Physical Infrastructure
       │
       ▼
      EC2
       │
       ▼
----------------------------
My Responsibility
----------------------------
       │
       ├── Operating System
       ├── Database
       ├── Patching
       ├── Backup
       ├── Scaling
       └── Availability
```

This provides control, but also creates more operational responsibility.

---

# 2. Amazon RDS

The second option introduced in the lesson is:

```text
Amazon Relational Database Service
              │
              ▼
             RDS
```

Instead of manually installing and managing the database on an EC2 instance, Amazon RDS provides a managed relational database service.

Conceptually:

```text
Application
     │
     ▼
Amazon RDS
     │
     ▼
Relational Database
```

AWS manages more of the underlying database infrastructure and administrative work.

---

# 🎯 Why Amazon RDS?

Running a database requires many operational tasks.

For example:

```text
Provision Infrastructure

Install Database

Patch Software

Perform Backups

Manage Availability

Scale Infrastructure
```

RDS reduces much of this administrative overhead.

The lesson describes RDS as taking care of areas such as:

```text
Database Deployment

Backups

Patching

Scaling

Availability Features

Database Infrastructure Management
```

This lets me focus more on:

```text
Application

Data

Schema

Queries

Database Usage
```

instead of maintaining the underlying database server.

---

# 🆚 Database on EC2 vs Amazon RDS

| Area                  | Database on EC2  | Amazon RDS                                    |
| --------------------- | ---------------- | --------------------------------------------- |
| Infrastructure        | AWS provides EC2 | AWS-managed RDS platform                      |
| OS Management         | Customer         | AWS handles underlying managed infrastructure |
| Database Installation | Customer         | Managed through RDS                           |
| OS Patching           | Customer         | Managed by AWS                                |
| Database Maintenance  | Customer         | More managed by RDS                           |
| Backups               | Customer manages | RDS provides managed capabilities             |
| Scaling               | Customer manages | RDS provides scaling capabilities             |
| Control               | More             | Less than self-managed EC2                    |
| Operational Effort    | Higher           | Lower                                         |

The simplest mental model:

```text
Database on EC2
       │
       ▼
More Control
       +
More Management


Amazon RDS
       │
       ▼
Managed Service
       +
Less Administrative Work
```

---

# 🗄️ RDS Database Engines

The lesson introduces these standard RDS database engines:

```text
Amazon RDS
│
├── MySQL
├── PostgreSQL
├── MariaDB
├── Microsoft SQL Server
├── IBM Db2
└── Oracle Database
```

It also introduces:

```text
Amazon Aurora
```

which is AWS's relational database engine compatible with:

```text
MySQL

and

PostgreSQL
```

The detailed features of RDS and Aurora come later.

---

# 🧩 Putting Everything Together

The complete picture is:

```text
Application
     │
     ▼
Relational Database
     │
     ├── Tables
     │
     ├── Rows
     │
     ├── Columns
     │
     ├── Primary Keys
     │
     ├── Foreign Keys
     │
     └── Relationships
            │
            ▼
           SQL
            │
       ┌────┴────┐
       │         │
      CRUD      ACID
       │         │
       └────┬────┘
            ▼
      Reliable Data
       Management
            │
            ▼
      Run on AWS
       ┌────┴────┐
       │         │
      EC2       RDS
       │         │
Self-Managed   Managed
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking a Row Is a Column

Remember:

```text
ROW
 =
Record


COLUMN
 =
Attribute / Field
```

---

## Mistake 2: Confusing Primary and Foreign Keys

```text
Primary Key
     │
     ▼
Uniquely identifies
record in its table


Foreign Key
     │
     ▼
References a record
in another table
```

---

## Mistake 3: Thinking Foreign Keys Must Be Unique

A customer can have multiple orders.

Therefore:

```text
Customer_ID = C002
```

can appear multiple times as a foreign key in the Orders table.

---

## Mistake 4: Confusing SQL with a Database

SQL is:

```text
Language
```

used to interact with relational databases.

It is not itself the database.

---

## Mistake 5: Confusing CRUD with ACID

```text
CRUD
   │
   ▼
What operations
can I perform?


ACID
   │
   ▼
How are transactions
processed reliably?
```

---

## Mistake 6: Thinking RDS Means I Have No Database Responsibilities

RDS removes a lot of infrastructure and administrative work.

But I still need to understand and manage areas such as:

```text
Data

Schema

Queries

Application Connectivity

Database Usage
```

---

# ❓ Interview Questions

### Q1. What is a relational database?

A relational database organizes structured data into tables that can be related to each other.

---

### Q2. What is a table?

A structure containing rows and columns.

---

### Q3. What does a row represent?

A:

```text
Record
```

---

### Q4. What does a column represent?

An:

```text
Attribute / Field
```

of a record.

---

### Q5. What is a primary key?

A column or attribute used to uniquely identify a record in a table.

---

### Q6. What is a foreign key?

A field that references a key in another table and helps create a relationship between the tables.

---

### Q7. What is a database schema?

The defined structure or architecture of the database, including tables, columns, keys, and relationships.

---

### Q8. What is SQL?

SQL stands for:

```text
Structured Query Language
```

and is used to query and manipulate relational database data.

---

### Q9. What does CRUD stand for?

```text
Create

Read

Update

Delete
```

---

### Q10. What does ACID stand for?

```text
Atomicity

Consistency

Isolation

Durability
```

---

### Q11. What does Atomicity mean?

A transaction either completes fully or does not complete at all.

---

### Q12. What does Consistency mean?

Transactions maintain the validity and integrity rules of the database.

---

### Q13. What does Isolation mean?

Concurrent transactions operate without incorrectly interfering with each other.

---

### Q14. What does Durability mean?

Once a transaction has been committed, its result remains recorded.

---

### Q15. What are the two approaches introduced for running relational databases on AWS?

```text
Database on EC2

Amazon RDS
```

---

### Q16. What is the main difference between running a database on EC2 and RDS?

With EC2, I manage much more of the operating system and database infrastructure.

With RDS, AWS manages more of the database platform and administrative operations.

---

### Q17. Which database engines are introduced as available through RDS?

The lesson introduces:

```text
MySQL

PostgreSQL

MariaDB

Microsoft SQL Server

IBM Db2

Oracle Database
```

---

### Q18. What is Amazon Aurora?

Aurora is an AWS relational database engine compatible with MySQL and PostgreSQL.

---

# 💡 Key Takeaways

* Applications need structured ways to store and manage data.
* Relational databases organize data into related tables.
* Tables contain rows and columns.
* A row represents a record.
* A column represents an attribute or field.
* Primary keys uniquely identify records.
* Foreign keys create relationships between tables.
* Joins allow information from related tables to be combined.
* The database schema defines the structure of the database.
* SQL is used to query and manipulate relational database data.
* CRUD stands for Create, Read, Update, and Delete.
* ACID stands for Atomicity, Consistency, Isolation, and Durability.
* ACID properties help provide reliable transaction processing.
* AWS allows me to run a relational database myself on EC2.
* Running a database on EC2 gives me more management responsibility.
* Amazon RDS provides a managed relational database service.
* RDS reduces much of the administrative overhead associated with running database infrastructure.
* The lesson introduces MySQL, PostgreSQL, MariaDB, Microsoft SQL Server, IBM Db2, and Oracle Database as RDS engines.
* Amazon Aurora is also introduced as an AWS relational database compatible with MySQL and PostgreSQL.

The easiest mental model is:

```text
RELATIONAL DATABASE
        │
        ▼
      TABLES
        │
   ┌────┴────┐
   ▼         ▼
 ROWS      COLUMNS
   │         │
   ▼         ▼
RECORDS   ATTRIBUTES
        │
        ▼
      KEYS
   ┌────┴────┐
   ▼         ▼
PRIMARY    FOREIGN
   │         │
   └────┬────┘
        ▼
RELATIONSHIPS
        │
        ▼
      JOINS
        │
        ▼
       SQL
```

And for AWS:

```text
Need Relational Database
          │
          ▼
   ┌──────┴──────┐
   │             │
   ▼             ▼
EC2 Database   Amazon RDS
   │             │
   ▼             ▼
More          More
Management    Managed
```

---

# 📚 Related Topics

* Amazon RDS
* Amazon Aurora
* MySQL
* PostgreSQL
* MariaDB
* Microsoft SQL Server
* Oracle Database
* IBM Db2
* SQL
* Database Schema
* Primary Keys
* Foreign Keys
* Database Joins
* CRUD Operations
* ACID Transactions
* Amazon EC2
