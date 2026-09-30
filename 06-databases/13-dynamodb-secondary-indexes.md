# 🔍 Amazon DynamoDB Secondary Indexes

> DynamoDB secondary indexes provide alternative keys for querying data efficiently. They allow applications to query attributes other than the base table's primary key without relying on expensive Scan operations.

---

# 📖 Overview

In the previous lesson, we looked at two ways of retrieving data from DynamoDB:

```text
Scan

Query
```

A Scan examines all items in a table or index.

A Query is more targeted because it retrieves items based on key values.

```text
SCAN
  │
  ▼
Read Through Table
  │
  ▼
Higher Read Capacity Consumption


QUERY
  │
  ▼
Use Key
  │
  ▼
Target Matching Items
```

Whenever possible, we want to design our DynamoDB tables so applications can use efficient **Query operations instead of Scan operations**.

This is where:

```text
Secondary Indexes
```

become important.

---

# 🎯 What Is a Secondary Index?

A secondary index is a data structure containing:

```text
Subset of Attributes
from Base Table

+

Alternative Key
```

The alternative key provides another way to query the data.

Conceptually:

```text
                 Base Table
                     │
                     │ Project Attributes
                     ▼
              Secondary Index
                     │
                     ▼
             Alternative Key
                     │
                     ▼
              Efficient Query
```

Applications can query the secondary index similarly to how they query the base table.

---

# 🏗️ Base Table

Every secondary index is associated with exactly one DynamoDB table.

That table is called the:

```text
Base Table
```

For example:

```text
Students Table
     │
     ├── StudentID
     ├── CourseCode
     ├── FirstName
     ├── LastName
     ├── Grade
     └── EnrollmentStatus
```

If we create an index from this table:

```text
Students Table
     │
     ▼
Secondary Index
```

the Students table is the **base table**.

---

# 🔑 Why Do We Need Secondary Indexes?

A DynamoDB Query operates using a partition key value.

Suppose the base table is designed like this:

```text
Partition Key = StudentID

Sort Key = CourseCode
```

This design allows efficient queries such as:

```text
StudentID = 1001
```

or queries involving the student's course information.

But what happens if the application needs to query by:

```text
Grade
```

or:

```text
EnrollmentStatus
```

instead?

Without another suitable key, we may need to:

```text
Scan Entire Table
       │
       ▼
Apply Filter
       │
       ▼
Return Matching Results
```

Secondary indexes provide alternative keys that can support these additional query patterns.

---

# 📦 Projected Attributes

When creating an index, we can specify which attributes from the base table should be:

```text
Projected
```

into the index.

Conceptually:

```text
Base Table

StudentID
CourseCode
FirstName
LastName
Grade
EnrollmentStatus
      │
      │ Projection
      ▼
Secondary Index

StudentID
Grade
EnrollmentStatus
```

The lesson introduces three projection choices:

```text
ALL

KEYS_ONLY

INCLUDE
```

These determine which attributes are copied into the secondary index.

---

# 🧩 Types of DynamoDB Secondary Indexes

DynamoDB provides two types of secondary indexes:

```text
Secondary Indexes
       │
       ├── Local Secondary Index
       │        LSI
       │
       └── Global Secondary Index
                GSI
```

They solve similar problems but have important architectural differences.

---

# PART 1: LOCAL SECONDARY INDEX (LSI)

# 📍 What Is a Local Secondary Index?

A:

```text
Local Secondary Index
        │
        ▼
       LSI
```

uses the:

```text
Same Partition Key
as Base Table

+

Different Sort Key
```

This provides another way to query items within the same partition key.

---

# 🏗️ LSI Architecture

Suppose the base table uses:

```text
Partition Key = StudentID

Sort Key = CourseCode
```

For example:

```text
Students Base Table

StudentID | CourseCode | Grade
----------|------------|------
1001      | SA001      | A
1001      | SA002      | B
1001      | SA003      | A
```

The normal key structure is:

```text
StudentID
    │
    ▼
Partition Key

CourseCode
    │
    ▼
Sort Key
```

Now suppose I want to efficiently query courses based on:

```text
Grade
```

I can create an LSI:

```text
LSI

Partition Key = StudentID

Sort Key = Grade
```

---

# 🔑 LSI Key Structure

The important relationship is:

```text
BASE TABLE

Partition Key
StudentID
     │
     └── Sort Key
         CourseCode


LOCAL SECONDARY INDEX

Partition Key
StudentID
     │
     └── Alternative Sort Key
         Grade
```

Notice:

```text
Partition Key
=
SAME
```

while:

```text
Sort Key
=
DIFFERENT
```

This is the key characteristic of an LSI.

---

# 🎯 LSI Query Example

Suppose the requirement is:

> Find all courses where Student 1001 received an A grade.

The table contains:

```text
StudentID | CourseCode | Grade
----------|------------|------
1001      | STAT101    | A
1001      | CLOUD101   | A
1001      | NET101     | B
1002      | CLOUD101   | A
```

Without the appropriate secondary index, finding items based on Grade may require broader scanning/filtering.

With an LSI:

```text
Partition Key
StudentID = 1001

+

LSI Sort Key
Grade = A
```

we can perform a Query.

```text
StudentID = 1001
       │
       ▼
Grade = A
       │
       ▼
Query LSI
       │
       ▼
Return Matching Courses
```

Result:

```text
1001 | STAT101  | A

1001 | CLOUD101 | A
```

The table does not need to be scanned completely.

---

# ⚡ LSI Efficiency Example

The lecture demonstrates this using the AWS Management Console.

Using Scan:

```text
Items Scanned
=
32

RCUs Consumed
=
2
```

The Scan must examine all 32 items before applying the filter.

Using the LSI:

```text
StudentID = 1001

Grade = A
```

the Query consumes:

```text
0.5 RCU
```

in the demonstration.

Conceptually:

```text
SCAN

32 Items
   │
   ▼
2 RCUs


LSI QUERY

StudentID = 1001
Grade = A
   │
   ▼
0.5 RCU
```

This demonstrates why an appropriate index can make queries much more efficient.

---

# ⏰ When Must an LSI Be Created?

An important LSI characteristic is:

```text
LSI
 │
 ▼
Must Be Created
When Table Is Created
```

The lesson emphasizes that an LSI cannot simply be added later to an existing table.

Therefore:

```text
Create DynamoDB Table
        │
        ├── Primary Key
        ├── Sort Key
        └── LSI
```

The LSI needs to be considered during the original table design.

---

# 🔢 Maximum Number of LSIs

The lesson states that a table can have:

```text
Maximum 5 LSIs
```

per table.

---

# ⚙️ LSI Capacity

An LSI shares capacity with its:

```text
Base Table
```

For provisioned capacity:

```text
Base Table
   │
   ├── RCUs
   └── WCUs
        │
        ▼
       LSI
```

The LSI uses the read and write capacity allocated to the base table rather than having its own independently provisioned capacity.

---

# 📦 LSI Projection Options

The lesson describes the following projection options:

```text
ALL

KEYS_ONLY

INCLUDE
```

This determines which attributes from the base table are available through the LSI.

---

# 🧠 LSI Mental Model

The easiest way to remember an LSI is:

```text
LOCAL SECONDARY INDEX

Same Partition Key
        │
        +
        │
Different Sort Key
```

For example:

```text
Base Table

StudentID + CourseCode


LSI

StudentID + Grade
```

---

# PART 2: GLOBAL SECONDARY INDEX (GSI)

# 🌍 What Is a Global Secondary Index?

A:

```text
Global Secondary Index
        │
        ▼
       GSI
```

provides an alternative key structure where the index can use a partition key and sort key different from those of the base table.

Conceptually:

```text
BASE TABLE

Partition Key = StudentID
Sort Key      = CourseCode


GLOBAL SECONDARY INDEX

Partition Key = EnrollmentStatus
Sort Key      = Grade
```

This enables completely different query patterns.

---

# 🔑 GSI Key Structure

Suppose our Students table contains:

```text
StudentID

CourseCode

FirstName

LastName

Grade

EnrollmentStatus
```

The base table might use:

```text
Partition Key
=
StudentID

Sort Key
=
CourseCode
```

But we want to query based on:

```text
EnrollmentStatus
```

We can create a GSI such as:

```text
Partition Key
=
EnrollmentStatus

Sort Key
=
Grade
```

Now the application can efficiently query using enrollment status.

---

# 🌍 Why Is It Called Global?

The lesson explains that a GSI is considered:

```text
Global
```

because queries against the index can span the base table across its partitions.

The GSI is stored in its:

```text
Own Partition Space
```

separate from the base table.

Conceptually:

```text
Base Table
Partition Space
      │
      │ Data Projected
      ▼
GSI
Separate Partition Space
```

The GSI can therefore scale separately from the base table.

---

# 🎯 GSI Use Case

Suppose we are building a student dashboard.

The requirement is:

> Show all inactive students.

The base table uses:

```text
StudentID
```

as the partition key.

But our requirement is based on:

```text
EnrollmentStatus
```

Without a GSI:

```text
Students Table
      │
      ▼
Scan Entire Table
      │
      ▼
Filter
EnrollmentStatus = Inactive
      │
      ▼
Return Results
```

This requires scanning the entire table.

---

# ⚡ Query Using a GSI

Instead, create a GSI:

```text
GSI Partition Key
=
EnrollmentStatus
```

Now the application can perform:

```text
Query GSI
     │
     ▼
EnrollmentStatus
=
Inactive
     │
     ▼
Return Inactive Students
```

No full table Scan is required.

---

# 📊 GSI with Sort Key

The lesson provides an example where the GSI uses:

```text
Partition Key
=
EnrollmentStatus

Sort Key
=
Grade
```

This allows queries such as:

> Find students with active enrollment who received an A grade.

```text
EnrollmentStatus = Active
          │
          +
          │
      Grade = A
          │
          ▼
       Query GSI
          │
          ▼
Matching Students
```

---

# 🧪 Console Example

The lecture demonstrates a Students table containing:

```text
15 Items
```

A full Scan consumes:

```text
17.5 RCUs
```

in the demonstration.

With the GSI, the application can query:

```text
EnrollmentStatus = Active

+

Grade = A
```

and consume significantly fewer RCUs because it does not need to scan the entire table.

---

# ⏰ When Can a GSI Be Created?

Unlike an LSI, a GSI can be created:

```text
When Table Is Created

OR

After Table Creation
```

This is an important difference.

```text
LSI
 │
 ▼
Creation Time Only


GSI
 │
 ├── Table Creation
 │
 └── Later
```

---

# 🔢 Maximum Number of GSIs

The lesson states that the default limit is:

```text
20 GSIs
per Base Table
```

---

# ⚙️ GSI Capacity

Unlike an LSI, a GSI uses its:

```text
Own Capacity
```

The lesson describes GSIs as requiring their own:

```text
RCUs

and

WCUs
```

when using provisioned capacity.

Conceptually:

```text
Base Table
│
├── Table RCUs
└── Table WCUs


GSI
│
├── GSI RCUs
└── GSI WCUs
```

The GSI therefore scales separately from the base table.

---

# 📦 GSI Projection Options

Like LSIs, GSIs can project:

```text
ALL

KEYS_ONLY

INCLUDE
```

attributes from the base table.

---

# 🔄 GSI Read Consistency

An important characteristic of GSIs is:

```text
GSI Reads
    │
    ▼
Eventually Consistent
```

The lesson emphasizes that Global Secondary Indexes do not provide strongly consistent reads.

---

# 🔒 LSI and Strong Consistency

The lesson contrasts this with LSIs.

If the application requires:

```text
Strongly Consistent Reads
```

the lesson points to:

```text
LSI
```

rather than a GSI.

This makes the timing of LSI creation important because:

```text
Need Strong Consistency
        │
        ▼
Need LSI
        │
        ▼
LSI Must Have Been Created
with the Table
```

---

# 🆚 LSI vs GSI

| Feature | Local Secondary Index | Global Secondary Index |
|---|---|---|
| Abbreviation | LSI | GSI |
| Partition Key | Same as base table | Can be different |
| Sort Key | Different from base table | Can be different |
| Creation | Only when table is created | At creation or later |
| Maximum Mentioned in Lesson | 5 | 20 default |
| Capacity | Shared with base table | Separate capacity |
| Partition Space | Associated with base table partition | Separate partition space |
| Strongly Consistent Reads | Available | Not available |
| Eventually Consistent Reads | Available | Available |
| Projection | ALL / KEYS_ONLY / INCLUDE | ALL / KEYS_ONLY / INCLUDE |
| Main Purpose | Alternative sort key for same partition key | Alternative partition and sort key |

---

# 🧠 LSI vs GSI Mental Model

The easiest way to remember the difference is:

```text
LSI

SAME Partition Key
DIFFERENT Sort Key
        │
        ▼
LOCAL


GSI

DIFFERENT Partition Key
DIFFERENT Sort Key
        │
        ▼
GLOBAL
```

And:

```text
LSI
 │
 ▼
Create With Table


GSI
 │
 ▼
Create Anytime
```

---

# 🏗️ Complete Example

Suppose our base table is:

```text
STUDENTS TABLE

Partition Key = StudentID
Sort Key      = CourseCode
```

We need three access patterns.

### Access Pattern 1

> Find all courses for Student 1001.

Use the base table:

```text
StudentID = 1001
```

---

### Access Pattern 2

> Find all courses where Student 1001 received an A.

Use an LSI:

```text
StudentID = 1001

Grade = A
```

Architecture:

```text
LSI

Partition Key
StudentID

+

Sort Key
Grade
```

---

### Access Pattern 3

> Find all inactive students.

Use a GSI:

```text
EnrollmentStatus = Inactive
```

Architecture:

```text
GSI

Partition Key
EnrollmentStatus

+

Sort Key
Grade
```

This demonstrates how secondary indexes support multiple application query patterns.

---

# ⚡ Why Secondary Indexes Improve Query Efficiency

Without a suitable key:

```text
Application
     │
     ▼
SCAN
     │
     ▼
Read Entire Table
     │
     ▼
Filter Results
     │
     ▼
Higher Read Consumption
```

With a suitable index:

```text
Application
     │
     ▼
QUERY INDEX
     │
     ▼
Use Index Key
     │
     ▼
Retrieve Matching Items
```

The goal is to avoid unnecessary table scans when the application has known access patterns.

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking an LSI Can Be Added Later

It cannot be added after table creation according to the lesson.

Remember:

```text
LSI
=
Create With Table
```

---

## Mistake 2: Thinking a GSI Must Be Created with the Table

A GSI can be created:

```text
During Table Creation

or

After Table Creation
```

---

## Mistake 3: Thinking an LSI Has a Different Partition Key

An LSI uses:

```text
Same Partition Key

Different Sort Key
```

---

## Mistake 4: Thinking a GSI Must Use the Base Table Partition Key

A GSI provides an alternative key structure.

It can use a different partition key and sort key.

---

## Mistake 5: Thinking LSIs Have Separate Capacity

LSIs share the capacity of the:

```text
Base Table
```

---

## Mistake 6: Thinking GSIs Share Base Table Capacity

The lesson describes GSIs as having their own:

```text
RCUs

and

WCUs
```

when using provisioned capacity.

---

## Mistake 7: Expecting Strongly Consistent Reads from a GSI

GSIs support:

```text
Eventually Consistent Reads
```

The lesson points to LSIs when strongly consistent reads are required.

---

## Mistake 8: Using Scan When an Index Can Support the Access Pattern

If the application frequently searches by:

```text
EnrollmentStatus
```

creating an appropriate GSI can provide a more efficient Query pattern than repeatedly scanning the table.

---

# ✅ Design Considerations

When designing DynamoDB tables, think about the queries the application needs to perform.

Ask:

```text
What attributes will users search by?

What is my partition key?

What is my sort key?

Do I need another sort key?

Do I need another partition key?

Do I need strongly consistent reads?
```

Then:

```text
Need Same Partition Key
but Different Sort Key?
        │
        ▼
       LSI


Need Different
Partition Key?
        │
        ▼
       GSI
```

---

# ❓ Interview Questions

### Q1. What is a DynamoDB secondary index?

A secondary index is a data structure containing a subset of attributes from the base table along with an alternative key that supports additional Query operations.

---

### Q2. Why do we use secondary indexes?

They allow applications to efficiently query data using attributes other than the base table's primary key.

---

### Q3. What is a base table?

The DynamoDB table from which a secondary index obtains its data.

---

### Q4. What are the two types of DynamoDB secondary indexes?

```text
Local Secondary Index (LSI)

Global Secondary Index (GSI)
```

---

### Q5. What is an LSI?

An LSI is a secondary index that uses:

```text
Same Partition Key
+
Different Sort Key
```

from the base table.

---

### Q6. When must an LSI be created?

```text
When the DynamoDB Table Is Created
```

---

### Q7. Can an LSI be added to an existing table later?

No, according to the lesson.

---

### Q8. How many LSIs can a table have according to the lesson?

```text
Maximum 5
```

---

### Q9. Does an LSI have its own provisioned capacity?

No.

It shares RCUs and WCUs with the base table.

---

### Q10. What projection options are available for an LSI?

```text
ALL

KEYS_ONLY

INCLUDE
```

---

### Q11. What is a GSI?

A Global Secondary Index provides an alternative key structure and can use a different partition key and sort key from the base table.

---

### Q12. Why is a GSI called global?

The lesson explains that queries against the GSI can span the base table across its partitions.

The GSI also maintains its own partition space.

---

### Q13. When can a GSI be created?

```text
During Table Creation

or

After Table Creation
```

---

### Q14. What is the default maximum number of GSIs mentioned in the lesson?

```text
20
```

per base table.

---

### Q15. Does a GSI have separate capacity?

Yes.

The lesson describes GSIs as having their own RCUs and WCUs when provisioned capacity is used.

---

### Q16. Does a GSI support strongly consistent reads?

No.

The lesson describes GSI reads as:

```text
Eventually Consistent
```

---

### Q17. Which secondary index should I consider if strongly consistent reads are required?

The lesson points to:

```text
LSI
```

for strongly consistent index reads.

---

### Q18. What is the key difference between an LSI and GSI?

```text
LSI
=
Same Partition Key
Different Sort Key


GSI
=
Alternative Partition Key
and Sort Key
```

---

### Q19. I need to query StudentID 1001 by Grade. Which index fits the example?

```text
LSI
```

because the StudentID partition key remains the same while Grade becomes an alternative sort key.

---

### Q20. I need to query all students by EnrollmentStatus. Which index fits the example?

```text
GSI
```

because EnrollmentStatus becomes an alternative partition key.

---

### Q21. Why might an index be preferable to a Scan?

A suitable index allows the application to use a Query instead of scanning the entire table and filtering the results afterward.

---

# 💡 Key Takeaways

- Secondary indexes provide additional DynamoDB query patterns.
- Every secondary index belongs to one base table.
- An index contains an alternative key and projected attributes from the base table.
- Secondary indexes help applications use Query instead of expensive Scan operations.
- DynamoDB supports Local Secondary Indexes and Global Secondary Indexes.
- An LSI uses the same partition key as the base table.
- An LSI uses a different sort key.
- An LSI must be created when the table is created.
- The lesson states that a table can have up to five LSIs.
- LSIs share capacity with the base table.
- A GSI can use a different partition key and sort key.
- A GSI can be created during or after table creation.
- The lesson states a default limit of 20 GSIs per base table.
- GSIs maintain separate partition space and scale separately.
- GSIs use their own RCUs and WCUs with provisioned capacity.
- GSIs provide eventually consistent reads.
- The lesson points to LSIs when strongly consistent index reads are required.
- Projection options include ALL, KEYS_ONLY, and INCLUDE.
- Good DynamoDB design starts by understanding the application's required access patterns.

The easiest way to remember everything is:

```text
                   SECONDARY INDEX
                         │
               ┌─────────┴─────────┐
               │                   │
               ▼                   ▼
              LSI                 GSI
               │                   │
               ▼                   ▼
       Same Partition       Different/Alternative
            Key               Partition Key
               │                   │
               ▼                   ▼
       Different Sort        Different/Alternative
            Key                 Sort Key
               │                   │
               ▼                   ▼
       Create With Table      Create Anytime
               │                   │
               ▼                   ▼
       Shares Capacity       Separate Capacity
               │                   │
               ▼                   ▼
       Strong Consistency    Eventual Consistency
         Available                Only
```

And for choosing between them:

```text
Need another way to query?
           │
           ▼
Do I keep the same partition key?
           │
      ┌────┴────┐
      │         │
     YES        NO
      │         │
      ▼         ▼
     LSI       GSI
```

---

# 📚 Related Topics

- Amazon DynamoDB
- DynamoDB Query
- DynamoDB Scan
- DynamoDB Primary Keys
- Partition Keys
- Sort Keys
- Composite Keys
- Local Secondary Indexes
- Global Secondary Indexes
- Projection Expressions
- Read Capacity Units
- Write Capacity Units
- Eventually Consistent Reads
- Strongly Consistent Reads
- DynamoDB Access Patterns