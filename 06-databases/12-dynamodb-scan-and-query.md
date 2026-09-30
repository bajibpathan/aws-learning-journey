# 🔍 Amazon DynamoDB Scan and Query Operations

> DynamoDB provides two main operations for retrieving multiple items from a table: **Scan** and **Query**. A Scan reads through the entire table or secondary index, while a Query retrieves items based on primary key values.

---

# 📖 Overview

Once data is stored in DynamoDB, applications need a way to retrieve it.

Consider a DynamoDB table containing student information:

```text
Students Table

StudentID | CourseID | FirstName | LastName | Module | Status
----------|----------|-----------|----------|--------|----------
1001      | SA001    | Alice     | Johnson  | EC2    | Enrolled
1001      | SA002    | Alice     | Johnson  | VPC    | Completed
1002      | SA001    | Bob       | Smith    | EC2    | Completed
1003      | SA003    | Carol     | Williams | RDS    | Enrolled
```

If this table contains a large amount of data, there are two important ways to retrieve items:

```text
DynamoDB
   │
   ├── Scan
   │
   └── Query
```

Although both retrieve data, they work very differently.

---

# 🔎 DynamoDB Scan

A:

```text
Scan
```

reads through:

```text
Every Item
in the Table

or

Secondary Index
```

Conceptually:

```text
              DynamoDB Table
                    │
                    ▼
                   Scan
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Item 1      Item 2      Item 3
        ▼           ▼           ▼
      Item 4      Item 5      Item 6
                    ...
                    │
                    ▼
               Read Everything
```

DynamoDB examines every item in the table or secondary index.

---

# 💰 Scan Operations Consume More Read Capacity

Because Scan reads through the complete dataset:

```text
Large Table
    │
    ▼
Scan Operation
    │
    ▼
Read Every Item
    │
    ▼
Higher RCU Consumption
```

Scan operations can therefore consume significant:

```text
Read Capacity Units
       │
       ▼
      RCUs
```

This becomes particularly important when tables contain large amounts of data.

---

# 📦 Scan Returns All Attributes by Default

By default, a Scan returns:

```text
All Attributes
```

from the items being scanned.

For example:

```text
StudentID

CourseID

FirstName

LastName

Module

Status
```

But sometimes the application only needs a few attributes.

For this, DynamoDB provides:

```text
Projection Expression
```

---

# 🎯 Projection Expression

A projection expression allows me to specify which attributes should be returned.

Suppose I only need:

```text
StudentID

CourseID

Status
```

Instead of returning:

```text
StudentID
CourseID
FirstName
LastName
Module
Status
```

I can request:

```text
StudentID
CourseID
Status
```

using a projection expression.

---

# ⚠️ Projection Does Not Reduce the Scan Work

An important lesson from the console demonstration is that returning fewer attributes does not mean DynamoDB scans fewer items.

For example:

```text
Full Scan

32 Items Scanned
        │
        ▼
All Attributes Returned
        │
        ▼
17.5 RCUs
```

The lesson then applies a projection expression:

```text
32 Items Scanned
        │
        ▼
Only Selected Attributes Returned
        │
        ▼
17.5 RCUs
```

The returned data is different, but DynamoDB still scans the complete dataset.

Therefore:

```text
Projection Expression
        │
        ▼
Controls Attributes Returned

NOT

Items Scanned
```

---

# 🎯 Filter Expression

A Scan can also use a:

```text
Filter Expression
```

to narrow down the results returned.

For example, suppose I only want students whose status is:

```text
Completed
```

Conceptually:

```text
Scan Table
    │
    ▼
Read Items
    │
    ▼
Apply Filter
    │
    ▼
Status = Completed
    │
    ▼
Return Matching Results
```

---

# ⚠️ Filtering Does Not Avoid the Full Scan

The important point is that DynamoDB still needs to scan the items.

Conceptually:

```text
DynamoDB Table
      │
      ▼
Scan Items
      │
      ▼
Consume Read Capacity
      │
      ▼
Apply Filter
      │
      ▼
Return Matching Items
```

Therefore, a filter expression controls:

```text
What Gets Returned
```

rather than preventing DynamoDB from scanning the table.

---

# 📦 Scan Request Size Limit

A single Scan request can retrieve a maximum of:

```text
1 MB
```

of data.

If the table contains more data than can be returned in that request:

```text
Large DynamoDB Table
        │
        ▼
     Scan
        │
        ▼
First 1 MB of Data
        │
        ▼
More Data Remaining
```

DynamoDB uses pagination to continue retrieving the remaining data.

---

# 📄 Scan Pagination

DynamoDB provides:

```text
LastEvaluatedKey
```

to determine whether additional data remains.

The basic logic is:

```text
Run Scan
    │
    ▼
Results Returned
    │
    ▼
Is LastEvaluatedKey Present?
       │
   ┌───┴───┐
   │       │
  YES      NO
   │       │
   ▼       ▼
More      Scan
Data      Complete
```

---

# 🔑 LastEvaluatedKey

If:

```text
LastEvaluatedKey
```

is present in the response:

```text
More Items Exist
```

If it is not present:

```text
No More Items
```

Therefore:

```text
LastEvaluatedKey Present
        │
        ▼
Continue Scanning


LastEvaluatedKey Missing
        │
        ▼
Scan Complete
```

---

# ▶️ ExclusiveStartKey

To retrieve the next set of results, take the:

```text
LastEvaluatedKey
```

from the previous response and use it as:

```text
ExclusiveStartKey
```

in the next Scan request.

The process becomes:

```text
Scan Request 1
      │
      ▼
Results
      │
      ▼
LastEvaluatedKey
      │
      ▼
Use as
ExclusiveStartKey
      │
      ▼
Scan Request 2
      │
      ▼
Next Results
```

This continues until there is no `LastEvaluatedKey`.

---

# 🔄 Complete Scan Pagination Flow

```text
Start Scan
    │
    ▼
Retrieve Up to 1 MB
    │
    ▼
LastEvaluatedKey?
    │
 ┌──┴──┐
 │     │
YES    NO
 │     │
 ▼     ▼
Use    Done
as
ExclusiveStartKey
 │
 ▼
Next Scan
 │
 ▼
Retrieve Next Page
```

---

# 📖 Scan Read Consistency

The lesson describes Scan operations as using:

```text
Eventually Consistent Reads
```

by default.

As discussed in the previous DynamoDB lesson, eventual consistency means a read might not immediately return the latest write.

---

# 🔒 Strongly Consistent Scan

If the latest data is required, the lesson describes setting:

```text
ConsistentRead = true
```

This changes the Scan to use:

```text
Strongly Consistent Reads
```

Conceptually:

```text
Default Scan
     │
     ▼
Eventually Consistent


ConsistentRead = true
     │
     ▼
Strongly Consistent
```

---

# 🔍 DynamoDB Query

A:

```text
Query
```

works differently from Scan.

Instead of reading every item, Query finds items based on:

```text
Primary Key Values
```

The application provides the partition key attribute and a value to search for.

---

# 🔑 Query Using the Partition Key

Suppose our table contains:

```text
StudentID
```

as the partition key.

To retrieve information for Alice:

```text
StudentID = 1001
```

I can query:

```text
DynamoDB Table
      │
      ▼
StudentID = 1001
      │
      ▼
Matching Items
```

DynamoDB retrieves the items associated with that partition key value.

---

# 🧩 Query Example

Suppose the table contains:

```text
StudentID | CourseID | Status
----------|----------|----------
1001      | SA001    | Enrolled
1001      | SA002    | Completed
1001      | SA003    | Enrolled
1002      | SA001    | Completed
1003      | SA002    | Enrolled
```

Query:

```text
StudentID = 1001
```

returns:

```text
1001 | SA001 | Enrolled
1001 | SA002 | Completed
1001 | SA003 | Enrolled
```

The items belonging to other partition key values are not part of the requested result.

---

# 🔑 Query with a Sort Key

A Query can also use the:

```text
Sort Key
```

to refine the results.

For example:

```text
Partition Key
StudentID = 1001

+

Sort Key
CourseID = SA001
```

Conceptually:

```text
StudentID = 1001
       │
       ▼
Matching Partition
       │
       ▼
CourseID = SA001
       │
       ▼
Specific Result
```

This allows a Query to narrow the results further.

---

# 🎯 Query Filter Expression

A Query can also use a:

```text
Filter Expression
```

to determine which items from the query results should be returned.

For example:

```text
StudentID = 1001
```

with:

```text
Status = Enrolled
```

Conceptually:

```text
Query
StudentID = 1001
       │
       ▼
Matching Items
       │
       ▼
Filter
Status = Enrolled
       │
       ▼
Return Results
```

---

# 📏 Limit Parameter

A Query can also use:

```text
Limit
```

to specify the maximum number of items to evaluate for the operation.

Conceptually:

```text
Query
   │
   ▼
StudentID = 1001
   │
   ▼
Limit
   │
   ▼
Maximum Number of Items
```

---

# 🧪 Console Example: Scan

The lesson demonstrates a DynamoDB table called:

```text
Students
```

containing:

```text
32 Items
```

A Scan is performed against all attributes.

The result is:

```text
Items Returned: 32

Items Scanned: 32

RCUs Consumed: 17.5
```

Conceptually:

```text
Students Table
      │
      ▼
Scan
      │
      ▼
32 Items Scanned
      │
      ▼
32 Items Returned
      │
      ▼
17.5 RCUs
```

---

# 🧪 Scan with Projection Expression

The demonstration then retrieves only selected attributes:

```text
StudentID

CourseCode

Status
```

The output contains fewer attributes.

However:

```text
RCUs Consumed
=
17.5
```

remains the same in the example.

Why?

Because DynamoDB still scans through the entire dataset.

```text
Projection Expression
       │
       ▼
Less Data Returned

BUT

Entire Dataset Scanned
       │
       ▼
Same Scan Consumption
in the Demonstration
```

---

# 🧪 Console Example: Query

The lesson then performs a Query using:

```text
StudentID = 1001
```

as the partition key value.

Only selected attributes are requested:

```text
StudentID

CourseCode

Status
```

Instead of scanning all 32 items, DynamoDB retrieves the items associated with:

```text
StudentID 1001
```

The lesson's demonstration shows:

```text
RCUs Consumed
=
0.5
```

compared with:

```text
Scan
=
17.5 RCUs
```

---

# 📊 Scan vs Query Example

From the lesson's console demonstration:

| Operation | Search Method | RCU Consumption |
|---|---|---:|
| Scan | Reads through entire table | 17.5 RCUs |
| Query | StudentID = 1001 | 0.5 RCU |

This demonstrates why Query can be much more efficient when the partition key for the required data is known.

---

# 🆚 Scan vs Query

| Area | Scan | Query |
|---|---|---|
| Data Access | Reads every item in table/index | Finds items using primary key values |
| Partition Key Required | No | Yes |
| Sort Key | Not required | Can refine results |
| Read Capacity | Can consume significant RCUs | More targeted |
| Projection Expression | Supported | Supported |
| Filter Expression | Supported | Supported |
| 1 MB Request Limit | Yes | Results can also require pagination |
| Best Fit | Need to examine broad dataset | Know the partition key |
| Efficiency | Generally more expensive | Generally more efficient for targeted access |

---

# 🧠 Scan Mental Model

Think of Scan as:

```text
DynamoDB Table
      │
      ▼
Read Everything
      │
      ▼
Find What I Need
```

For example:

```text
Student Table
      │
      ▼
Check Alice
Check Bob
Check Carol
Check David
Check Emma
Check Frank
...
      │
      ▼
Return Matches
```

---

# 🎯 Query Mental Model

Think of Query as:

```text
I Know the Key
      │
      ▼
Find Matching Partition
      │
      ▼
Retrieve Matching Items
```

For example:

```text
StudentID = 1001
       │
       ▼
Find Student 1001
       │
       ▼
Return Courses
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Scan and Query Are the Same

They retrieve data differently.

```text
SCAN
 │
 ▼
Read Entire Table


QUERY
 │
 ▼
Use Primary Key Value
```

---

## Mistake 2: Using Scan When the Partition Key Is Known

If I know:

```text
StudentID = 1001
```

a Query can target that key instead of scanning the entire table.

---

## Mistake 3: Thinking Projection Expression Reduces Items Scanned

Projection controls:

```text
Attributes Returned
```

It does not mean the Scan avoids reading the items.

The lesson's demonstration shows:

```text
All Attributes
=
17.5 RCUs


Selected Attributes
=
17.5 RCUs
```

for the same Scan.

---

## Mistake 4: Thinking a Filter Makes Scan Behave Like Query

A filter can reduce:

```text
Results Returned
```

but the Scan still reads through the table.

---

## Mistake 5: Forgetting the 1 MB Scan Limit

A single Scan request retrieves a maximum of:

```text
1 MB
```

according to the lesson.

Larger results require pagination.

---

## Mistake 6: Forgetting LastEvaluatedKey

If the Scan response contains:

```text
LastEvaluatedKey
```

there are more results to retrieve.

---

## Mistake 7: Forgetting ExclusiveStartKey

For the next Scan:

```text
Previous LastEvaluatedKey
          │
          ▼
New ExclusiveStartKey
```

---

## Mistake 8: Assuming Scan Uses Strong Consistency by Default

The lesson describes the default as:

```text
Eventually Consistent
```

To request strongly consistent reads:

```text
ConsistentRead = true
```

---

# ✅ Best Practices

- Prefer Query when the partition key for the required data is known.
- Understand that Scan reads through the entire table or secondary index.
- Be careful with Scan operations on large tables because they can consume significant read capacity.
- Use projection expressions when only specific attributes need to be returned.
- Do not assume projection expressions reduce the number of items scanned.
- Use filter expressions when returned results need additional filtering.
- Understand that filtering does not prevent Scan from reading the underlying items.
- Handle pagination when Scan results exceed the request size limit.
- Check `LastEvaluatedKey` to determine whether additional results remain.
- Use the previous `LastEvaluatedKey` as `ExclusiveStartKey` for the next request.
- Use strongly consistent reads when the latest successful writes must be reflected.

---

# ❓ Interview Questions

### Q1. What is the difference between DynamoDB Scan and Query?

A Scan reads every item in a table or secondary index.

A Query retrieves items based on primary key values.

---

### Q2. Which operation generally consumes more read capacity?

```text
Scan
```

because it reads through the complete table or index.

---

### Q3. What does a Scan return by default?

```text
All Attributes
```

from the scanned items.

---

### Q4. How can I return only selected attributes?

Use a:

```text
Projection Expression
```

---

### Q5. Does a projection expression reduce the number of items scanned?

No.

It controls which attributes are returned, but the Scan still reads through the items.

---

### Q6. What is a filter expression?

A filter expression narrows the results returned by the operation based on specified conditions.

---

### Q7. Does a Scan filter prevent DynamoDB from scanning the table?

No.

The table is scanned and the filter determines which results are returned.

---

### Q8. What is the maximum amount of data returned by a single Scan request according to the lesson?

```text
1 MB
```

---

### Q9. How do I know whether additional Scan results exist?

Check for:

```text
LastEvaluatedKey
```

in the response.

---

### Q10. What does the presence of LastEvaluatedKey mean?

```text
More Results Exist
```

---

### Q11. What does the absence of LastEvaluatedKey mean?

```text
Scan Complete
```

---

### Q12. How do I retrieve the next page?

Use the previous:

```text
LastEvaluatedKey
```

as the:

```text
ExclusiveStartKey
```

for the next Scan request.

---

### Q13. What read consistency does Scan use by default according to the lesson?

```text
Eventually Consistent Reads
```

---

### Q14. How can I request strongly consistent Scan results?

Set:

```text
ConsistentRead = true
```

---

### Q15. What information must be supplied for a Query?

The lesson describes supplying:

```text
Partition Key Attribute

+

Partition Key Value
```

---

### Q16. Can a sort key be used with Query?

Yes.

The sort key can refine the Query results further.

---

### Q17. Can Query use filter expressions?

Yes.

A filter expression can determine which items from the Query results are returned.

---

### Q18. What is the Limit parameter?

It defines the maximum number of items for the operation as described in the lesson.

---

### Q19. What did the Scan consume in the lesson's console example?

```text
17.5 RCUs
```

for the demonstrated 32-item table.

---

### Q20. Did selecting fewer attributes reduce the Scan's RCU consumption in the demonstration?

No.

It remained:

```text
17.5 RCUs
```

---

### Q21. What did the Query consume in the console demonstration?

The Query for:

```text
StudentID = 1001
```

consumed:

```text
0.5 RCU
```

in the example.

---

### Q22. When should I think about Query instead of Scan?

When I know the partition key value for the data I need.

---

# 💡 Key Takeaways

- DynamoDB provides Scan and Query operations for retrieving data.
- Scan reads every item in a table or secondary index.
- Scan can consume significant read capacity.
- Scan returns all attributes by default.
- Projection expressions allow selected attributes to be returned.
- Projection expressions do not prevent Scan from reading the underlying items.
- Filter expressions narrow the results returned.
- Filtering does not eliminate the underlying Scan.
- A single Scan request can return up to 1 MB according to the lesson.
- `LastEvaluatedKey` indicates that additional results remain.
- Use `LastEvaluatedKey` as `ExclusiveStartKey` in the next request.
- Scan uses eventually consistent reads by default.
- `ConsistentRead = true` requests strongly consistent reads.
- Query retrieves items using primary key values.
- A Query requires a partition key value.
- A sort key can further refine Query results.
- Query supports filter expressions.
- Query can use a Limit parameter.
- Query is more targeted than Scan when the required partition key is known.
- In the lesson's console example, Scan consumed 17.5 RCUs while the targeted Query consumed 0.5 RCU.

The easiest way to remember the difference is:

```text
SCAN
 │
 ▼
"I don't know exactly where it is"
 │
 ▼
Read Through Everything
 │
 ▼
More Read Capacity


QUERY
 │
 ▼
"I know the partition key"
 │
 ▼
Go to Matching Data
 │
 ▼
More Targeted Retrieval
```

And remember the Scan pagination flow:

```text
SCAN
 │
 ▼
Up to 1 MB
 │
 ▼
LastEvaluatedKey?
 │
 ├── NO ─────► DONE
 │
 └── YES
       │
       ▼
Use as ExclusiveStartKey
       │
       ▼
Next Scan
       │
       ▼
Repeat
```

---

# 📚 Related Topics

- Amazon DynamoDB
- DynamoDB Tables
- DynamoDB Items and Attributes
- Partition Keys
- Sort Keys
- Composite Keys
- DynamoDB Scan
- DynamoDB Query
- Projection Expressions
- Filter Expressions
- Pagination
- LastEvaluatedKey
- ExclusiveStartKey
- Read Capacity Units
- Eventually Consistent Reads
- Strongly Consistent Reads
- DynamoDB Secondary Indexes