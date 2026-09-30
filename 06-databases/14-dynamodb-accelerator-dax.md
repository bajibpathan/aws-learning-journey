# ⚡ Amazon DynamoDB Accelerator (DAX)

> Amazon DynamoDB Accelerator (DAX) is an in-memory caching service for DynamoDB that can reduce response times from milliseconds to microseconds by serving frequently accessed data from a cache instead of repeatedly retrieving it from the DynamoDB table.

---

# 📖 Overview

Amazon DynamoDB is designed for high performance and scalability.

Typical DynamoDB response times can be:

```text
Single-Digit Milliseconds
```

For many applications, this is already very fast.

However, some applications require even faster response times:

```text
Milliseconds
     │
     ▼
Microseconds
```

For these workloads, DynamoDB provides:

```text
DynamoDB Accelerator
        │
        ▼
       DAX
```

DAX provides an in-memory caching layer between the application and DynamoDB.

---

# 🎯 Why Do We Need DAX?

Without DAX:

```text
Application
     │
     ▼
DynamoDB
     │
     ▼
Retrieve Data
     │
     ▼
Application
```

Every request that needs data is sent to DynamoDB.

DynamoDB is already designed to provide single-digit millisecond performance, but some applications may require even lower latency.

With DAX:

```text
Application
     │
     ▼
    DAX
     │
     ▼
Cached Data
```

Frequently accessed data can be returned directly from the in-memory cache.

This can reduce response times from:

```text
Milliseconds
     │
     ▼
Microseconds
```

---

# 🧠 What Is DAX?

DAX stands for:

```text
DynamoDB Accelerator
```

It is an:

```text
In-Memory Cache
       │
       ▼
for DynamoDB
```

The basic architecture is:

```text
Application
     │
     ▼
DAX Cluster
     │
     ▼
DynamoDB Table
```

The application communicates with DAX, and DAX communicates with the backend DynamoDB table when required.

---

# 🏗️ DAX Cluster Architecture

DAX is deployed as a:

```text
DAX Cluster
```

inside a VPC.

The lesson describes the cluster as containing:

```text
Primary Node

+

Read Replicas
```

Conceptually:

```text
                 DAX Cluster
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Primary     Replica     Replica
         Node        Node        Node
```

The cluster architecture provides failover capability between the primary node and replicas.

This helps provide:

```text
High Availability
```

---

# 🌐 DAX and the VPC

The DAX cluster is deployed inside a:

```text
VPC
```

The application should also run within the same VPC.

For example:

```text
                    VPC
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
   EC2 Application           DAX Cluster
                                  │
                                  ▼
                              DynamoDB
```

The lesson emphasizes selecting the correct VPC when creating the DAX cluster.

If a VPC is not explicitly selected, the cluster may be deployed into the default VPC.

---

# 🖥️ Application Architecture

Suppose an application is running on Amazon EC2.

The architecture becomes:

```text
                    VPC
                     │
              EC2 Application
                     │
                     ▼
                 DAX Client
                     │
                     ▼
                DAX Endpoint
                     │
                     ▼
                 DAX Cluster
                     │
                     ▼
              DynamoDB Table
```

The application does not simply start using DAX automatically.

It needs to be configured to communicate with the DAX cluster.

---

# 🔌 DAX Client

The application uses a:

```text
DAX Client
```

The DAX client knows how to connect to the:

```text
DAX Endpoint
```

Conceptually:

```text
Application
     │
     ▼
DAX Client
     │
     ▼
DAX Endpoint
     │
     ▼
DAX Cluster
```

The application can then perform supported DynamoDB data operations through DAX.

---

# 🔍 Cache Hit and Cache Miss

The two most important caching concepts are:

```text
Cache Hit

Cache Miss
```

These determine whether DAX can return data directly from memory or needs to retrieve it from DynamoDB.

---

# ❌ Cache Miss

Suppose the application requests some data for the first time.

```text
Application
     │
     ▼
DAX
     │
     ▼
Is Data Cached?
     │
     ▼
    NO
```

This is called a:

```text
CACHE MISS
```

DAX then sends the request to the backend DynamoDB table.

---

# 🔄 Cache Miss Workflow

The complete process is:

```text
Application
     │
     ▼
DAX Cluster
     │
     ▼
Cache Miss
     │
     ▼
DynamoDB
     │
     ▼
Retrieve Data
     │
     ▼
DAX Cache
     │
     ▼
Store Result
     │
     ▼
Return Data
     │
     ▼
Application
```

The important point is that DAX does two things after retrieving the information:

```text
Return Data to Application

+

Store Data in Cache
```

The cached information can then be used for future requests.

---

# ✅ Cache Hit

Suppose the application requests the same data again.

```text
Application
     │
     ▼
DAX Cluster
     │
     ▼
Is Data Cached?
     │
     ▼
    YES
```

This is called a:

```text
CACHE HIT
```

DAX can return the information directly from memory.

```text
Application
     │
     ▼
DAX
     │
     ▼
Cache Hit
     │
     ▼
Return Cached Data
```

There is no need to retrieve the same information from the backend DynamoDB table for that request.

---

# ⚡ Cache Hit Performance

This is where DAX provides its major performance benefit.

```text
DynamoDB
     │
     ▼
Milliseconds


DAX Cache Hit
     │
     ▼
Microseconds
```

For workloads requiring extremely low read latency, retrieving cached information can therefore provide a significant improvement.

---

# 🔄 Complete DAX Read Flow

The overall process looks like:

```text
                    Application
                         │
                         ▼
                       DAX
                         │
                         ▼
                  Data in Cache?
                         │
                ┌────────┴────────┐
                │                 │
               YES                NO
                │                 │
                ▼                 ▼
            CACHE HIT         CACHE MISS
                │                 │
                ▼                 ▼
          Return Cached        DynamoDB
              Data                │
                                  ▼
                            Retrieve Data
                                  │
                                  ▼
                              Cache Data
                                  │
                                  ▼
                          Return to Application
```

---

# 🗄️ Two Types of DAX Cache

The lesson describes two cache types maintained by DAX:

```text
DAX
 │
 ├── Item Cache
 │
 └── Query Cache
```

They cache different DynamoDB operations.

---

# 1️⃣ Item Cache

The:

```text
Item Cache
```

stores results from:

```text
GetItem

BatchGetItem
```

operations.

Conceptually:

```text
Application
     │
     ▼
GetItem
     │
     ▼
DAX Item Cache
```

If the requested item is already cached:

```text
Item Cache
    │
    ▼
Cache Hit
    │
    ▼
Return Item
```

Otherwise:

```text
Item Cache
    │
    ▼
Cache Miss
    │
    ▼
DynamoDB
```

---

# 2️⃣ Query Cache

DAX also maintains a:

```text
Query Cache
```

The query cache stores results from:

```text
Query

Scan
```

operations.

Conceptually:

```text
Application
     │
     ▼
Query / Scan
     │
     ▼
DAX Query Cache
```

The results are cached based on the parameters used for the request.

---

# 🔑 Query Cache Parameters

Suppose an application sends:

```text
Query
Partition Key = StudentID 1001
```

DAX uses the request parameters to identify the cached result.

Conceptually:

```text
Query Parameters
      │
      ▼
DAX Query Cache
      │
      ▼
Matching Result Set?
```

If the same request has already been cached:

```text
YES
 │
 ▼
Cache Hit
 │
 ▼
Return Results
```

---

# ❌ Query Cache Miss

If DAX cannot find a matching cached result:

```text
Query / Scan
      │
      ▼
DAX Query Cache
      │
      ▼
No Matching Result
      │
      ▼
Cache Miss
```

DAX forwards the request to DynamoDB.

```text
DAX
 │
 ▼
DynamoDB
 │
 ▼
Process Request
 │
 ▼
Return Result Set
 │
 ▼
DAX Query Cache
 │
 ▼
Store Result
 │
 ▼
Application
```

---

# 🔄 Query Cache Workflow

```text
Application
     │
     ▼
Query / Scan
     │
     ▼
DAX Query Cache
     │
     ▼
Matching Parameters?
     │
 ┌───┴───┐
 │       │
YES      NO
 │       │
 ▼       ▼
Return   DynamoDB
Cached      │
Result      ▼
         Execute
         Request
            │
            ▼
        Return Data
            │
            ▼
        Cache Result
            │
            ▼
        Application
```

---

# 📖 DAX and Read Consistency

The lesson describes DAX retrieving query and scan results from DynamoDB using:

```text
Eventually Consistent Reads
```

This is important when considering the type of application data being cached.

---

# ⏱️ Time to Live (TTL)

Cached data should not necessarily remain in the cache forever.

DAX allows a:

```text
Time to Live
     │
     ▼
    TTL
```

to be configured for cached results.

The TTL determines how long the cached information remains valid.

Conceptually:

```text
Data Added to Cache
        │
        ▼
      TTL Starts
        │
        ▼
 Cached for Defined
     Time Period
        │
        ▼
      TTL Expires
        │
        ▼
Cache Entry Expires
```

---

# 🎯 Why Use TTL?

Suppose data is cached for a defined amount of time.

```text
Cached Data
    │
    ▼
TTL
    │
    ▼
Expiration
```

The TTL prevents cached information from remaining indefinitely.

After expiration, a future request may need to retrieve the data again from DynamoDB.

---

# ✍️ DAX Write-Through Behavior

DAX also provides:

```text
Write-Through
```

behavior for certain DynamoDB write operations.

The lesson identifies:

```text
BatchWriteItem

UpdateItem

DeleteItem

PutItem
```

as write-through operations.

---

# 🔄 What Does Write-Through Mean?

Suppose the application writes data.

Instead of only updating DynamoDB:

```text
Application
     │
     ▼
DynamoDB
```

DAX maintains the relevant cache information as part of the write-through process.

Conceptually:

```text
Application
     │
     ▼
    DAX
     │
     ▼
Write Operation
     │
     ├────────► DynamoDB Table
     │
     └────────► DAX Cache
```

The purpose is to keep the cache aligned with data changes performed through supported DAX write operations.

---

# 📝 Write-Through Operations

The lesson identifies these operations:

| Operation | Write-Through |
|---|---|
| `PutItem` | Yes |
| `UpdateItem` | Yes |
| `DeleteItem` | Yes |
| `BatchWriteItem` | Yes |

A useful memory aid is:

```text
PUT

UPDATE

DELETE

BATCH WRITE

        │
        ▼
   Write-Through
```

---

# 🚫 Operations DAX Does Not Handle

DAX is designed for application data operations.

It does **not** handle DynamoDB table-management operations such as:

```text
CreateTable

UpdateTable
```

These are administrative operations rather than normal cached application data operations.

---

# 🛠️ Table Management Architecture

If an application or management process needs to perform table administration:

```text
Application / Management Process
              │
              ▼
           DynamoDB
              │
              ▼
        CreateTable
        UpdateTable
        etc.
```

It should connect directly to DynamoDB rather than sending these operations through DAX.

Therefore:

```text
DATA OPERATIONS
      │
      ▼
     DAX


TABLE MANAGEMENT
      │
      ▼
DynamoDB Directly
```

---

# 🏗️ Complete DAX Architecture

Putting everything together:

```text
                         VPC
                          │
                    EC2 Application
                          │
                          ▼
                      DAX Client
                          │
                          ▼
                     DAX Endpoint
                          │
                          ▼
                    ┌───────────┐
                    │DAX Cluster│
                    └─────┬─────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
          Item Cache              Query Cache
              │                       │
              └───────────┬───────────┘
                          │
                    Cache Miss?
                          │
                          ▼
                     DynamoDB
```

Inside the DAX cluster:

```text
                 DAX Cluster
                     │
         ┌───────────┼───────────┐
         │           │           │
         ▼           ▼           ▼
      Primary      Replica     Replica
        Node         Node        Node
```

---

# 🆚 DynamoDB Without DAX vs With DAX

| Area | DynamoDB Only | DynamoDB + DAX |
|---|---|---|
| Typical Read Response | Single-digit milliseconds | Microseconds for cached data |
| Caching | No DAX caching layer | In-memory DAX cache |
| Repeated Reads | DynamoDB processes request | Can be served from cache |
| Item Cache | No | Yes |
| Query Cache | No | Yes |
| TTL | Not applicable to DAX cache | Configurable for cached data |
| Write-Through | Not through DAX | Supported for specified write operations |
| Table Management | Direct DynamoDB | Still performed directly against DynamoDB |

---

# 🆚 DAX Item Cache vs Query Cache

| Cache | Used For |
|---|---|
| Item Cache | `GetItem`, `BatchGetItem` |
| Query Cache | `Query`, `Scan` |

Easy memory aid:

```text
Get Individual Items
        │
        ▼
    ITEM CACHE


Query / Scan Results
        │
        ▼
    QUERY CACHE
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking DAX Replaces DynamoDB

DAX is a caching layer.

```text
Application
     │
     ▼
    DAX
     │
     ▼
DynamoDB
```

The backend data still resides in DynamoDB.

---

## Mistake 2: Thinking DAX Automatically Works Without Application Changes

The application needs:

```text
DAX Client
```

and needs to connect to the DAX endpoint.

---

## Mistake 3: Forgetting That DAX Is Deployed in a VPC

The DAX cluster is deployed inside a VPC.

The lesson recommends explicitly selecting the appropriate VPC rather than allowing it to be placed into the default VPC.

---

## Mistake 4: Confusing Item Cache and Query Cache

Remember:

```text
GetItem
BatchGetItem
     │
     ▼
Item Cache


Query
Scan
     │
     ▼
Query Cache
```

---

## Mistake 5: Thinking a Cache Miss Is an Error

A cache miss simply means the requested data is not currently cached.

```text
Cache Miss
    │
    ▼
DynamoDB
    │
    ▼
Retrieve Data
    │
    ▼
Populate Cache
```

---

## Mistake 6: Forgetting TTL

Cached data can have a:

```text
TTL
```

that controls how long it remains in the cache.

---

## Mistake 7: Sending Table Management Operations Through DAX

Operations such as:

```text
CreateTable

UpdateTable
```

must be performed directly against DynamoDB rather than through DAX.

---

# ❓ Interview Questions

### Q1. What is DAX?

DAX stands for:

```text
DynamoDB Accelerator
```

It provides an in-memory caching layer for DynamoDB.

---

### Q2. Why would I use DAX?

To provide faster access to frequently requested DynamoDB data, particularly when an application requires response times measured in microseconds.

---

### Q3. What performance improvement does the lesson associate with DAX?

```text
DynamoDB
Single-Digit Milliseconds

        ↓

DAX
Microseconds
```

---

### Q4. Where is a DAX cluster deployed?

Inside a:

```text
VPC
```

---

### Q5. What happens if I don't select a VPC when creating the DAX cluster according to the lesson?

The lesson states that AWS can deploy it into the default VPC.

---

### Q6. What components can a DAX cluster contain?

```text
Primary Node

+

Read Replicas
```

---

### Q7. Why have replicas in the DAX cluster?

The cluster architecture provides failover capability and helps support high availability.

---

### Q8. How does an application connect to DAX?

Using a:

```text
DAX Client
```

that connects to the DAX endpoint.

---

### Q9. What is a cache miss?

A cache miss occurs when the requested information is not available in the DAX cache.

DAX then retrieves it from DynamoDB.

---

### Q10. What happens after a cache miss?

```text
DAX
 │
 ▼
DynamoDB
 │
 ▼
Retrieve Data
 │
 ▼
Store in Cache
 │
 ▼
Return to Application
```

---

### Q11. What is a cache hit?

A cache hit occurs when the requested information is already available in DAX.

DAX returns the cached information directly to the application.

---

### Q12. What are the two types of cache maintained by DAX?

```text
Item Cache

Query Cache
```

---

### Q13. Which operations use the item cache?

```text
GetItem

BatchGetItem
```

---

### Q14. Which operations use the query cache?

```text
Query

Scan
```

---

### Q15. How does DAX identify cached Query or Scan results?

The query cache stores results based on the request's parameter values.

---

### Q16. What is TTL in DAX?

TTL stands for:

```text
Time to Live
```

and controls how long cached data remains before expiring.

---

### Q17. What is write-through caching?

Write-through means supported writes are sent to DynamoDB while DAX also maintains the relevant cached data.

---

### Q18. Which write-through operations are mentioned in the lesson?

```text
PutItem

UpdateItem

DeleteItem

BatchWriteItem
```

---

### Q19. Can DAX perform CreateTable or UpdateTable operations?

No.

The lesson states that DynamoDB table-management operations must be performed directly against DynamoDB.

---

### Q20. What is the easiest way to remember DAX?

```text
DynamoDB
     │
     ▼
Milliseconds


Need Faster Reads?
     │
     ▼
    DAX
     │
     ▼
In-Memory Cache
     │
     ▼
Microseconds
```

---

# 💡 Key Takeaways

- DAX stands for DynamoDB Accelerator.
- DAX provides in-memory caching for DynamoDB.
- DynamoDB typically provides single-digit millisecond performance.
- DAX can provide microsecond response times for cached data.
- DAX is deployed as a cluster inside a VPC.
- A DAX cluster can contain a primary node and read replicas.
- The cluster architecture provides failover capability.
- Applications use a DAX client to connect to the DAX endpoint.
- A cache miss causes DAX to retrieve the data from DynamoDB and cache the result.
- A cache hit allows DAX to return cached information directly.
- DAX maintains an item cache and a query cache.
- The item cache stores results from `GetItem` and `BatchGetItem`.
- The query cache stores results from `Query` and `Scan`.
- Query cache results are associated with their request parameters.
- DAX supports TTL for cached information.
- `PutItem`, `UpdateItem`, `DeleteItem`, and `BatchWriteItem` are described as write-through operations.
- DAX does not handle DynamoDB table-management operations such as `CreateTable` and `UpdateTable`.
- Table-management operations must connect directly to DynamoDB.

The simplest DAX mental model is:

```text
FIRST REQUEST

Application
     │
     ▼
    DAX
     │
     ▼
CACHE MISS
     │
     ▼
DynamoDB
     │
     ▼
Retrieve Data
     │
     ▼
Cache Result
     │
     ▼
Application
```

Then:

```text
REPEATED REQUEST

Application
     │
     ▼
    DAX
     │
     ▼
CACHE HIT
     │
     ▼
Microsecond Response

No DynamoDB Read Required
for the Cached Result
```

And remember the two caches:

```text
                   DAX
                    │
           ┌────────┴────────┐
           │                 │
           ▼                 ▼
       ITEM CACHE        QUERY CACHE
           │                 │
     GetItem             Query
     BatchGetItem        Scan
```

---

# 📚 Related Topics

- Amazon DynamoDB
- DynamoDB Accelerator (DAX)
- In-Memory Caching
- Cache Hit
- Cache Miss
- DAX Item Cache
- DAX Query Cache
- GetItem
- BatchGetItem
- Query
- Scan
- Time to Live (TTL)
- Write-Through Caching
- DynamoDB Read Performance
- Amazon VPC