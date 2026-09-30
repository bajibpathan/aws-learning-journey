# 🌍 DynamoDB Streams, Global Tables, and Time to Live (TTL)

> Amazon DynamoDB provides additional features for reacting to data changes, building globally distributed applications, and automatically removing expired data. Three important features are DynamoDB Streams, Global Tables, and Time to Live (TTL).

---

# 📖 Overview

This note covers three DynamoDB capabilities:

```text id="ix7h5k"
DynamoDB
   │
   ├── Streams
   │     └── React to table changes
   │
   ├── Global Tables
   │     └── Multi-Region, multi-active tables
   │
   └── Time to Live (TTL)
         └── Automatically expire old items
```

Each solves a different problem.

| Feature | Primary Purpose |
|---|---|
| DynamoDB Streams | Capture item-level changes |
| Global Tables | Multi-Region read/write access |
| TTL | Automatically expire unwanted items |

---

# PART 1: DYNAMODB STREAMS

# 🌊 What Are DynamoDB Streams?

DynamoDB Streams allow applications to monitor changes occurring in a DynamoDB table.

A stream maintains a:

```text id="pq31ha"
Time-Ordered Sequence
of
Item-Level Modifications
```

When items in the table change, those modifications can be recorded in the stream.

Examples include:

```text id="nq6gta"
INSERT

UPDATE

DELETE
```

This allows applications to react to changes as they occur.

---

# 🏗️ Basic Architecture

Consider a DynamoDB table:

```text id="w5zy1n"
DynamoDB Table
│
├── Item 1
├── Item 2
├── Item 3
└── Item 4
```

If an item changes:

```text id="i3i2v3"
DynamoDB Table
      │
      │ Item Modified
      ▼
DynamoDB Stream
      │
      ▼
Change Record
```

The change is recorded in the stream.

---

# ⏱️ Stream Retention

DynamoDB Streams maintain change information for:

```text id="krbep4"
24 Hours
```

This is a rolling period.

Conceptually:

```text id="xbjic3"
New Changes
     │
     ▼
DynamoDB Stream
     │
     ▼
Retained for 24 Hours
     │
     ▼
Older Records Removed
```

As new changes occur, new records are added while older records eventually expire.

---

# 🔄 What Changes Can Be Captured?

Streams can capture item-level changes such as:

```text id="m6iq4c"
New Item Added

Existing Item Updated

Item Deleted

Attribute Modified
```

For example:

```text id="fj1bgg"
Original Item

StudentID = 1001
Name      = Alice
Course    = AWS
Grade     = A
```

Suppose `Grade` is removed.

That modification can be captured by DynamoDB Streams.

---

# 📸 Stream Record Options

When enabling DynamoDB Streams, we need to decide how much information about the changed item should be recorded.

The lesson introduces four options:

```text id="y1s4bn"
KEYS_ONLY

NEW_IMAGE

OLD_IMAGE

NEW_AND_OLD_IMAGES
```

---

# 1️⃣ KEYS_ONLY

With:

```text id="dlrmf9"
KEYS_ONLY
```

the stream records only the key attributes of the modified item.

For example:

```text id="ve9d4f"
Partition Key = StudentID 1001

Sort Key = CourseCode AWS101
```

The stream knows **which item changed**, but it does not contain the complete before-and-after item information.

The application may need to use the key to retrieve additional information from the DynamoDB table.

---

# 2️⃣ NEW_IMAGE

With:

```text id="wd1jph"
NEW_IMAGE
```

the stream records the complete item:

```text id="ep8l7u"
After Modification
```

For example, before:

```text id="7p5h2a"
StudentID = 1001
Name      = Alice
Course    = AWS
Grade     = A
```

Suppose `Grade` is deleted.

The new image becomes:

```text id="3v0p6e"
StudentID = 1001
Name      = Alice
Course    = AWS
```

The stream records what the item looks like **after the change**.

---

# 3️⃣ OLD_IMAGE

With:

```text id="u0i1rb"
OLD_IMAGE
```

the stream records the complete item:

```text id="gxthdm"
Before Modification
```

Using the previous example:

```text id="whbqsa"
StudentID = 1001
Name      = Alice
Course    = AWS
Grade     = A
```

would be stored as the old image.

This allows an application to see what the item looked like before the modification.

---

# 4️⃣ NEW_AND_OLD_IMAGES

With:

```text id="cgvxpt"
NEW_AND_OLD_IMAGES
```

both versions are captured.

```text id="90vh5k"
BEFORE

StudentID = 1001
Name      = Alice
Course    = AWS
Grade     = A

        │
        ▼
     CHANGE
        │
        ▼

AFTER

StudentID = 1001
Name      = Alice
Course    = AWS
```

The application can compare the two versions and determine exactly what changed.

---

# 📊 Stream Record Comparison

| Option | Information Recorded |
|---|---|
| `KEYS_ONLY` | Partition key and sort key |
| `NEW_IMAGE` | Item after modification |
| `OLD_IMAGE` | Item before modification |
| `NEW_AND_OLD_IMAGES` | Both before and after versions |

A useful memory aid is:

```text id="6iyq8e"
KEYS_ONLY
   │
   └── Which item?


NEW_IMAGE
   │
   └── What does it look like now?


OLD_IMAGE
   │
   └── What did it look like before?


NEW_AND_OLD_IMAGES
   │
   └── What exactly changed?
```

---

# ⚡ Event-Driven Processing

Simply recording changes is useful, but the real benefit comes from **reacting to those changes**.

DynamoDB Streams can trigger event-driven processing.

A common example is:

```text id="xkox7j"
DynamoDB Table
      │
      │ Change
      ▼
DynamoDB Stream
      │
      ▼
AWS Lambda
      │
      ▼
Business Logic
```

The Lambda function can process the change and perform another action.

---

# 🛒 Example: Shopping Cart

Suppose a customer removes a product from a shopping cart.

```text id="bplxdq"
Customer
   │
   ▼
Remove Product
   │
   ▼
DynamoDB Table Updated
   │
   ▼
DynamoDB Stream
   │
   ▼
Lambda Function
```

The application could then perform business logic based on that change.

For example:

```text id="qf9jwc"
Product Removed
      │
      ▼
Wait / Process
      │
      ▼
Send Follow-Up Email
```

The business might later remind the customer:

> You previously showed interest in this product. Are you still interested?

This demonstrates how Streams can support event-driven application logic.

---

# 🎯 DynamoDB Streams Use Cases

Based on the lesson, Streams can be useful for:

```text id="o8n8qy"
Monitoring Data Changes

Event-Driven Processing

Triggering Lambda Functions

Tracking Updates

Reacting to Deletes

Recording Before/After Changes

Audit-Type Processing
```

---

# 🧠 Streams Mental Model

Remember:

```text id="s6hx0c"
DynamoDB Table
      │
      ▼
Something Changes
      │
      ▼
DynamoDB Stream
      │
      ▼
24-Hour Change Record
      │
      ▼
Lambda / Application Logic
```

---

# PART 2: DYNAMODB GLOBAL TABLES

# 🌍 What Are DynamoDB Global Tables?

DynamoDB Global Tables provide a:

```text id="v3zq7u"
Multi-Region

+

Multi-Active
```

database architecture.

The lesson also describes this as:

```text id="n0ce32"
Multi-Master
```

This means applications can read and write to DynamoDB tables in multiple AWS Regions.

---

# 🆚 Traditional Single-Writer Architecture

The lesson contrasts this with a database architecture where there is a single writable database.

Conceptually:

```text id="51evfk"
             Primary Database
                  READ/WRITE
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
       Read Replica        Read Replica
          READ                READ
```

Writes still need to reach the writable database.

For global applications, this can become challenging when users are distributed across different geographic locations.

---

# 🌎 DynamoDB Global Architecture

With DynamoDB Global Tables:

```text id="szh33r"
                Global Table
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
     London        Sydney          US
      Table         Table         Table
       │             │             │
   READ/WRITE    READ/WRITE    READ/WRITE
```

Each participating Region has a replica table.

Applications can interact with a nearby regional replica.

---

# ⚡ Multi-Active Architecture

A Global Table is a collection of:

```text id="ff0g03"
Multi-Active Tables
```

Each replica can support:

```text id="c7o09z"
Reads

and

Writes
```

This helps provide fast localized application access.

For example:

```text id="smpc46"
London Users
     │
     ▼
London Table


Sydney Users
     │
     ▼
Sydney Table


US Users
     │
     ▼
US Table
```

Users can interact with a regional table rather than sending all database operations to one distant Region.

---

# 🔄 Automatic Multi-Region Replication

When data is written in one Region:

```text id="oyrkhc"
Write in London
      │
      ▼
London Table
      │
      ├────────► Sydney Table
      │
      └────────► US Table
```

DynamoDB automatically replicates the changes to the replica tables in the other configured Regions.

The lesson describes this replication as occurring within approximately:

```text id="bov3yt"
1 Second
```

---

# 🔄 Replication Example

Suppose:

```text id="8d84no"
London
   │
   ▼
Customer Updates Profile
   │
   ▼
London DynamoDB Table
```

The change is then replicated:

```text id="bl81ge"
London
   │
   ├────────► Sydney
   │
   └────────► US
```

The regional tables eventually receive the updated information.

---

# 📖 Global Tables and Consistency

The lesson describes Global Tables as using:

```text id="jrr9lt"
Eventually Consistent
Replication
```

Because multiple Regions can accept writes, there is a possibility that conflicting updates could occur.

For example:

```text id="up7c79"
London
   │
   └── Update Item X


Sydney
   │
   └── Update Item X
```

If these modifications occur around the same time, DynamoDB needs a way to reconcile them.

---

# 🏆 Conflict Resolution: Last Writer Wins

The lesson describes the reconciliation mechanism as:

```text id="nkh4f4"
Last Writer Wins
```

Conceptually:

```text id="ph2q9c"
Write A
London
   │
   ├────────────┐
   │            │
   ▼            │
Global Table    │
                │
Write B         │
Sydney          │
   │            │
   └────────────┘
        │
        ▼
Determine Latest Write
        │
        ▼
Last Writer Wins
        │
        ▼
Replicate Result
```

The winning version is then propagated to the other replicas.

---

# 🏗️ Converting an Existing Table

The lesson also explains that an existing DynamoDB table can be converted into a Global Table.

For example:

```text id="7v67pz"
Stage 1

London
  │
  ▼
Single DynamoDB Table
```

As the business grows:

```text id="x4q7f8"
Stage 2

              Global Table
                   │
         ┌─────────┼─────────┐
         ▼         ▼         ▼
      London    Sydney       US
```

This allows the application architecture to expand into additional Regions.

---

# 🌍 Regional Migration Use Case

Global Tables can also help when moving an application from one Region to another.

For example:

```text id="ew5r94"
Original Region
London
   │
   ▼
Create Global Table
   │
   ▼
Add Sydney Replica
   │
   ▼
Replicate Data
   │
   ▼
Move Application
to Sydney
```

Once the migration is complete, the previous regional configuration could be decommissioned if appropriate.

---

# ⚙️ Global Table Capacity

The lesson mentions capacity planning for replica tables.

When using Global Tables, capacity requirements must account for replicated reads and writes across participating Regions.

The important concept is:

```text id="4pscf7"
Local Database Activity

+

Replication Activity

=

Global Table Capacity
```

Each participating Region needs sufficient capacity for the workload it handles.

---

# 🎯 Global Table Use Cases

Global Tables are useful when applications require:

```text id="7fczmw"
Multi-Region Presence

Localized Reads

Localized Writes

Global Applications

Regional Expansion

Regional Migration
```

---

# 🧠 Global Tables Mental Model

Remember:

```text id="whmkgn"
              DYNAMODB GLOBAL TABLE
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
       Region A      Region B      Region C
      READ/WRITE    READ/WRITE    READ/WRITE
         │             │             │
         └─────────────┼─────────────┘
                       │
                       ▼
             Automatic Replication
                       │
                       ▼
                Last Writer Wins
```

---

# PART 3: DYNAMODB TIME TO LIVE (TTL)

# ⏱️ What Is DynamoDB TTL?

DynamoDB provides a feature called:

```text id="rvjcyq"
Time to Live
     │
     ▼
    TTL
```

TTL allows unwanted items to expire automatically after a specified time.

Instead of keeping old data indefinitely:

```text id="qg4b94"
Create Item
    │
    ▼
Store Forever
```

we can define an expiration time:

```text id="q7q9uj"
Create Item
    │
    ▼
TTL Attribute
    │
    ▼
Expiration Time
    │
    ▼
Automatic Deletion
```

---

# 🎯 Why Use TTL?

Over time, a DynamoDB table may accumulate data that is no longer required.

Examples include:

```text id="5bxg30"
Expired Session Data

Temporary Application Data

Old Records

Data No Longer Required
for Compliance or Business Purposes
```

TTL provides a cost-effective mechanism for removing such items automatically.

---

# 🏷️ TTL Attribute

To use TTL, we define an attribute that contains the item's expiration time.

For example:

```text id="6buz2s"
SessionID | UserID | Data | ExpirationTime
----------|--------|------|---------------
ABC123    | 1001   | ...  | 1790000000
XYZ456    | 1002   | ...  | 1791000000
```

The expiration attribute is used by DynamoDB to determine when the item is no longer required.

---

# 🕐 Unix Epoch Time

The TTL value uses:

```text id="zgfkv7"
Unix Epoch Time
```

The background process compares:

```text id="fh24o2"
Current Time
     │
     ▼
TTL Attribute
```

If the expiration timestamp is older than the current time:

```text id="rqwl9h"
TTL < Current Time
       │
       ▼
Item Expired
```

the item becomes eligible for deletion.

---

# 🔄 TTL Processing

The process can be visualized as:

```text id="3qcx3k"
DynamoDB Item
     │
     ▼
TTL Attribute
     │
     ▼
Compare with Current Time
     │
     ▼
Has TTL Expired?
     │
 ┌───┴───┐
 │       │
NO      YES
 │       │
 ▼       ▼
Keep    Eligible
Item    for Deletion
```

---

# ⚠️ TTL Deletion Is Not Immediate

An important point is that reaching the expiration timestamp does not mean the item disappears at that exact second.

The lesson states that expired items are typically deleted:

```text id="ufphl2"
Within a Few Days
of Expiration
```

Therefore:

```text id="h60y5o"
TTL Reached
    │
    ▼
Item Expired
    │
    ▼
Eligible for Deletion
    │
    ▼
DynamoDB Background Process
    │
    ▼
Item Deleted
```

Applications should therefore not assume that an expired item will immediately disappear from the table.

---

# 💰 TTL and Write Capacity

For a standard DynamoDB table, the lesson explains that TTL deletion does not consume:

```text id="nm19bm"
Write Capacity Units
        │
        ▼
       WCUs
```

DynamoDB performs the expiration deletion automatically.

This makes TTL a cost-effective mechanism for removing old data.

---

# 🌊 TTL with DynamoDB Streams

TTL deletion activity can also be recorded through:

```text id="72cfda"
DynamoDB Streams
```

The architecture becomes:

```text id="xhd0sg"
DynamoDB Item
      │
      ▼
TTL Expires
      │
      ▼
Item Deleted
      │
      ▼
DynamoDB Stream
      │
      ▼
Deletion Record
      │
      ▼
Application / Lambda
```

This connects the two features we discussed earlier.

---

# 🔄 Example: Processing TTL Deletes

Suppose an application stores temporary session information.

```text id="yp3cr0"
Session Created
     │
     ▼
DynamoDB
     │
     ▼
TTL Attribute
     │
     ▼
Session Expires
     │
     ▼
DynamoDB Deletes Item
     │
     ▼
DynamoDB Stream
     │
     ▼
Process Deletion Event
```

The stream can be used to monitor when the delete occurs and trigger additional business logic.

---

# 🛒 Session Data Use Case

One good TTL use case is:

```text id="83aox6"
Session Data
```

Suppose an application stores temporary user sessions:

```text id="bhth1j"
SessionID
UserID
LoginTime
ExpirationTime
```

The session might only need to exist for a limited period.

Instead of manually deleting old sessions:

```text id="ib26q8"
Session
   │
   ▼
TTL Expires
   │
   ▼
Automatic Deletion
```

DynamoDB can handle the cleanup automatically.

---

# 📋 Compliance Use Case

The lesson also mentions regulatory or compliance requirements.

For example:

```text id="cvd6qv"
Data Created
    │
    ▼
Required Retention Period
    │
    ▼
TTL Reached
    │
    ▼
Data Expires
```

TTL can therefore help remove information that no longer needs to be retained.

---

# 🌍 TTL with Global Tables

TTL behaves differently when used with:

```text id="k6av4g"
Global Tables
```

Suppose the item expires in one Region:

```text id="k3ap1e"
Region A
   │
   ▼
TTL Delete
```

That delete must also be replicated to the other replica tables:

```text id="3us0x1"
             TTL Delete
                 │
                 ▼
              Region A
                 │
        ┌────────┴────────┐
        ▼                 ▼
     Region B          Region C
  Replicated Delete  Replicated Delete
```

The lesson emphasizes that while the original TTL deletion does not consume standard write capacity on the source table, replicated deletes associated with Global Tables can incur capacity usage and charges in replica Regions.

---

# 🔗 How These Three Features Work Together

Streams, Global Tables, and TTL solve different problems, but they can also interact.

Consider:

```text id="6q7igx"
DynamoDB Table
      │
      ├── Global Table
      │      │
      │      ▼
      │   Replicate Data
      │   Across Regions
      │
      ├── TTL
      │      │
      │      ▼
      │   Expire Old Items
      │
      └── Streams
             │
             ▼
        Capture Changes
             │
             ▼
           Lambda
```

For example:

1. An application stores session data.
2. The data is replicated through a Global Table.
3. TTL eventually expires the session.
4. DynamoDB deletes the expired item.
5. The deletion can appear in DynamoDB Streams.
6. An application or Lambda function can react to the deletion.

---

# 📊 Feature Comparison

| Feature | Streams | Global Tables | TTL |
|---|---|---|---|
| Primary Purpose | Track changes | Multi-Region database | Expire old data |
| Main Behavior | Records modifications | Replicates tables | Deletes expired items |
| Multi-Region | No | Yes | Can work with Global Tables |
| Retention/Timing | 24-hour stream records | Continuous replication | Delete after expiration |
| Event Driven | Yes | Not primary purpose | Can integrate with Streams |
| Lambda Integration | Common use case | Not focus of lesson | Through Streams |
| Key Concept | Ordered change records | Multi-active | Expiration timestamp |

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking DynamoDB Streams Store Changes Permanently

The lesson states:

```text id="04cwbf"
Stream Retention
=
24 Hours
```

Streams should not be treated as permanent change-history storage.

---

## Mistake 2: Confusing NEW_IMAGE and OLD_IMAGE

Remember:

```text id="p28pua"
OLD_IMAGE
=
Before Change


NEW_IMAGE
=
After Change
```

---

## Mistake 3: Thinking KEYS_ONLY Contains the Complete Item

`KEYS_ONLY` contains only the key attributes identifying the modified item.

---

## Mistake 4: Thinking Global Tables Have One Writable Region

Global Tables are:

```text id="24gj5f"
Multi-Active
```

Replica tables in participating Regions can accept reads and writes.

---

## Mistake 5: Forgetting Global Table Conflict Resolution

The lesson describes the conflict-resolution mechanism as:

```text id="fj83fm"
Last Writer Wins
```

---

## Mistake 6: Thinking TTL Deletes Items Exactly at the Expiration Time

TTL expiration makes the item eligible for deletion.

The lesson states that actual deletion typically occurs within a few days.

---

## Mistake 7: Forgetting TTL Uses Unix Epoch Time

The TTL attribute uses:

```text id="yyt6hc"
Unix Epoch Time
```

for the expiration timestamp.

---

## Mistake 8: Assuming TTL Deletes Are Free Everywhere in a Global Table

The original TTL deletion does not consume standard WCUs on the source table as described in the lesson, but replicated deletes in Global Table replica Regions can incur capacity usage and charges.

---

# ❓ Interview Questions

### Q1. What is DynamoDB Streams?

DynamoDB Streams provides a time-ordered sequence of item-level modifications made to a DynamoDB table.

---

### Q2. How long does DynamoDB Streams retain change records?

```text id="pxex1r"
24 Hours
```

---

### Q3. What changes can appear in a DynamoDB Stream?

Changes such as:

```text id="h1z7b7"
INSERT

UPDATE

DELETE
```

can be captured.

---

### Q4. What are the four stream record options discussed?

```text id="ab6sjm"
KEYS_ONLY

NEW_IMAGE

OLD_IMAGE

NEW_AND_OLD_IMAGES
```

---

### Q5. What does KEYS_ONLY store?

Only the key attributes identifying the modified item.

---

### Q6. What does NEW_IMAGE store?

The entire item as it appears after the modification.

---

### Q7. What does OLD_IMAGE store?

The entire item as it appeared before the modification.

---

### Q8. When would I use NEW_AND_OLD_IMAGES?

When the application needs both the before and after versions to determine what changed.

---

### Q9. Can DynamoDB Streams trigger Lambda functions?

Yes. Streams can support event-driven processing where changes trigger Lambda functions and application logic.

---

### Q10. What are DynamoDB Global Tables?

Global Tables provide a multi-Region, multi-active DynamoDB architecture.

---

### Q11. Can applications write to multiple Global Table Regions?

Yes.

The lesson describes Global Tables as:

```text id="1iuv5m"
Multi-Active / Multi-Master
```

---

### Q12. How quickly does the lesson say writes are replicated?

The lesson describes writes as being replicated to other Regions within approximately:

```text id="8t0bcd"
1 Second
```

---

### Q13. What consistency model does the lesson associate with Global Tables?

```text id="rtlj1c"
Eventually Consistent
```

---

### Q14. How are write conflicts handled according to the lesson?

Using:

```text id="ssphfr"
Last Writer Wins
```

reconciliation.

---

### Q15. Can an existing DynamoDB table become a Global Table?

Yes.

The lesson describes converting an existing table as an application expands into additional Regions.

---

### Q16. What is DynamoDB TTL?

TTL automatically expires items that are no longer required based on a per-item expiration timestamp.

---

### Q17. What time format does TTL use?

```text id="38hd6x"
Unix Epoch Time
```

---

### Q18. Is an item deleted immediately when its TTL expires?

No.

The lesson states that expired items are generally deleted within a few days after expiration.

---

### Q19. Does a standard TTL deletion consume WCUs?

The lesson states that the automatic TTL deletion itself does not consume write throughput on the standard table.

---

### Q20. Can TTL deletions appear in DynamoDB Streams?

Yes.

TTL deletion activity can be captured through DynamoDB Streams.

---

### Q21. What is a good TTL use case?

```text id="g7amko"
Temporary Session Data
```

that is no longer needed after a defined period.

---

### Q22. What should I remember about TTL and Global Tables?

TTL deletes replicated to other Global Table Regions can consume replicated capacity and incur charges.

---

# 💡 Key Takeaways

## DynamoDB Streams

```text id="9c3dsw"
Table Changes
     │
     ▼
DynamoDB Streams
     │
     ▼
24-Hour Ordered Change Records
     │
     ▼
Lambda / Business Logic
```

Remember:

- Tracks item-level modifications.
- Records are retained for 24 hours.
- Supports `KEYS_ONLY`.
- Supports `NEW_IMAGE`.
- Supports `OLD_IMAGE`.
- Supports `NEW_AND_OLD_IMAGES`.
- Can trigger event-driven application logic.

---

## DynamoDB Global Tables

```text id="ysn0wp"
              GLOBAL TABLE
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Region A   Region B   Region C
       R/W        R/W        R/W
        │          │          │
        └──────────┼──────────┘
                   ▼
             Replication
                   │
                   ▼
          Last Writer Wins
```

Remember:

- Multi-Region.
- Multi-active.
- Reads and writes can occur in multiple Regions.
- Data is automatically replicated.
- Replication is eventually consistent as described in the lesson.
- Conflicts use last-writer-wins reconciliation.
- Existing tables can be expanded into Global Tables.

---

## DynamoDB TTL

```text id="37wmq1"
Item
 │
 ▼
TTL Attribute
 │
 ▼
Unix Epoch Timestamp
 │
 ▼
Expiration Reached
 │
 ▼
Eligible for Deletion
 │
 ▼
Automatic Cleanup
```

Remember:

- TTL automatically expires unwanted items.
- Uses a per-item expiration attribute.
- Uses Unix epoch time.
- Expired items are not necessarily deleted immediately.
- The lesson says deletion generally occurs within a few days.
- Standard TTL deletion does not consume WCUs.
- TTL deletion events can appear in DynamoDB Streams.
- Replicated TTL deletes in Global Tables can incur capacity usage and charges.

---

# 🧠 Final Mental Model

Think of the three services this way:

```text id="g59gwz"
Something CHANGED?
       │
       ▼
DynamoDB Streams


Need data in MULTIPLE REGIONS?
       │
       ▼
DynamoDB Global Tables


Data no longer NEEDED?
       │
       ▼
DynamoDB TTL
```

Or even simpler:

```text id="86f6j9"
STREAMS
=
REACT


GLOBAL TABLES
=
REPLICATE


TTL
=
EXPIRE
```

---

# 📚 Related Topics

- Amazon DynamoDB
- DynamoDB Streams
- AWS Lambda
- Event-Driven Architecture
- Stream Records
- KEYS_ONLY
- NEW_IMAGE
- OLD_IMAGE
- NEW_AND_OLD_IMAGES
- DynamoDB Global Tables
- Multi-Region Architecture
- Multi-Active Databases
- Eventual Consistency
- Last Writer Wins
- DynamoDB Time to Live
- Unix Epoch Time
- DynamoDB Session Data