# ⚡ Amazon ElastiCache

> Amazon ElastiCache is an in-memory data store that improves application performance by caching frequently accessed data, reducing repeated queries to backend databases, and providing fast access to temporary data such as application sessions.

---

# 📖 Overview

In a traditional application architecture, application data is stored in a backend database.

When users request information:

```text
User
  │
  ▼
Application
  │
  ▼
Database Query
  │
  ▼
Retrieve Data
  │
  ▼
Application
  │
  ▼
User
```

Every time the application needs information, it may need to query the database.

For a small number of users, this may not be a major concern.

But imagine:

```text
Thousands of Users
        │
        ▼
    Application
        │
        ▼
Thousands of Queries
        │
        ▼
      Database
```

Many of these users may even be requesting the **same information repeatedly**.

This creates unnecessary work for the database.

A caching layer can help solve this problem.

---

# 🎯 The Problem: Repeated Database Queries

Consider an application where many users request the same content.

Without caching:

```text
User 1 ─────┐
User 2 ─────┤
User 3 ─────┤
User 4 ─────┼────► Application ─────► Database
User 5 ─────┤
   ...       │
User N ─────┘
```

The application may repeatedly send similar queries to the backend database.

Database queries consume:

```text
CPU

Memory

Processing Time
```

As the number of users increases:

```text
More Users
    │
    ▼
More Queries
    │
    ▼
More Database Processing
    │
    ▼
Higher Resource Consumption
```

We therefore need a mechanism that can reduce unnecessary database queries while improving application response times.

---

# ⚡ What Is Amazon ElastiCache?

Amazon ElastiCache provides an:

```text
In-Memory Data Store
```

for applications.

Instead of retrieving frequently accessed information from the backend database every time, the application can retrieve it from the cache.

```text
Application
     │
     ▼
ElastiCache
     │
     ▼
Frequently Accessed Data
```

Because the information is stored in memory, applications can retrieve cached information quickly.

---

# 🔑 Key-Value Storage

The lesson describes ElastiCache as storing information using:

```text
Key → Value
```

A key identifies the object, and the value contains the corresponding information.

For example:

```text
Key                  Value

product:1001    →    Product Information

user:123        →    User Information

session:456     →    Session Data

cart:789        →    Shopping Cart Data
```

Conceptually:

```text
Key
 │
 ▼
Lookup
 │
 ▼
Value
```

This provides a simple way for applications to quickly retrieve cached information.

---

# 📖 Read-Heavy Workloads

ElastiCache is particularly useful for:

```text
Read-Heavy Workloads
```

Suppose thousands of users request the same information.

Without caching:

```text
3,000 Users
     │
     ▼
Application
     │
     ▼
Repeated Queries
     │
     ▼
Database
```

With caching:

```text
3,000 Users
     │
     ▼
Application
     │
     ▼
ElastiCache
     │
     ▼
Cached Data
```

The backend database does not need to repeatedly process the same query.

---

# 🚀 Performance Benefits

Introducing ElastiCache can provide:

```text
Higher Performance

Lower Query Latency

Fewer Database Queries

Reduced Database Processing
```

Instead of:

```text
Application
     │
     ▼
Database
     │
     ▼
Process Query
     │
     ▼
Return Result
```

the application may retrieve the information directly from:

```text
Application
     │
     ▼
ElastiCache
     │
     ▼
Return Cached Result
```

---

# 🧠 Application Awareness

An important point from the lesson is that the application must be designed to work with the cache.

ElastiCache does not simply replace the database.

The application needs logic to determine:

```text
Is Data Available
in the Cache?
```

If yes:

```text
Read from Cache
```

If no:

```text
Read from Database
```

Therefore, developers need to include caching behavior in the application logic.

---

# 🗄️ ElastiCache Engines

The lesson introduces the following engines available with ElastiCache:

```text
Valkey

Memcached

Redis OSS
```

These provide the in-memory caching capability used by applications.

---

# 🏗️ Basic ElastiCache Architecture

Without ElastiCache:

```text
User
 │
 ▼
Application
 │
 ▼
Database
```

With ElastiCache:

```text
User
 │
 ▼
Application
 │
 ├────────► ElastiCache
 │
 │
 └────────► Database
```

The application checks the cache before querying the database.

---

# 🎯 Cache Hit and Cache Miss

Two important caching concepts are:

```text
Cache Hit

Cache Miss
```

Understanding these concepts helps explain how the application interacts with ElastiCache.

---

# ❌ Cache Miss

A:

```text
Cache Miss
```

occurs when the requested information is **not available in the cache**.

For example, imagine a user requesting a web page for the first time.

```text
User
 │
 ▼
Application
 │
 ▼
ElastiCache
 │
 ▼
Data Available?
 │
 ▼
NO
```

This is a:

```text
CACHE MISS
```

The application must then retrieve the information from the backend database.

---

# 🔄 Cache Miss Workflow

When a cache miss occurs:

```text
User
 │
 ▼
Application
 │
 ▼
Check Cache
 │
 ▼
CACHE MISS
 │
 ▼
Query Database
 │
 ▼
Retrieve Data
 │
 ▼
Update Cache
 │
 ▼
Return Data to User
```

The important part is:

```text
Database Result
      │
      ▼
Store in Cache
```

Now the information is available in the cache for future requests.

---

# ✅ Cache Hit

Suppose another user requests the same information.

The application checks the cache:

```text
User
 │
 ▼
Application
 │
 ▼
ElastiCache
 │
 ▼
Data Available?
 │
 ▼
YES
```

This is called a:

```text
CACHE HIT
```

The application can retrieve the information directly from the cache.

---

# ⚡ Cache Hit Workflow

```text
User
 │
 ▼
Application
 │
 ▼
Check Cache
 │
 ▼
CACHE HIT
 │
 ▼
Retrieve Cached Data
 │
 ▼
Return to User
```

The application does **not need to query the backend database again** for that request.

---

# 🔁 Complete Caching Flow

The complete workflow looks like this:

```text
                     User
                       │
                       ▼
                  Application
                       │
                       ▼
                 Check Cache
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
         CACHE HIT         CACHE MISS
              │                 │
              ▼                 ▼
       Return Cached         Database
           Data                 │
                                ▼
                         Retrieve Data
                                │
                                ▼
                          Update Cache
                                │
                                ▼
                         Return to User
```

This is the core idea behind using ElastiCache.

---

# 📊 Why Does Caching Matter?

Consider a single user requesting some data.

```text
1 User
   │
   ▼
Database Query
```

The additional database workload may not be significant.

Now imagine:

```text
3,000 Users
```

or:

```text
30,000 Users
```

requesting similar information.

Without caching:

```text
30,000 Users
      │
      ▼
30,000 Requests
      │
      ▼
Database Queries
```

With caching:

```text
30,000 Users
      │
      ▼
Application
      │
      ▼
ElastiCache
      │
      ▼
Cached Content
```

The cache can serve repeated requests without sending every request back to the database.

---

# 💰 Database Query Cost

Database queries have a cost in terms of:

```text
Time

CPU

Memory

Database Processing
```

Repeatedly performing the same query wastes database resources.

Caching frequently requested information reduces that unnecessary processing.

---

# 🛒 Session Data Use Case

Another important use case introduced in the lesson is storing:

```text
Session Data
```

Consider an e-commerce application.

A user visits the website and starts adding products to a shopping cart.

```text
User
 │
 ▼
E-Commerce Application
 │
 ▼
Shopping Cart
 │
 ├── Product A
 ├── Product B
 └── Product C
```

This shopping cart represents information associated with the user's current session.

---

# 🧺 Shopping Cart Session

Instead of constantly storing and retrieving temporary shopping cart information from the backend database, the application can store the session state in ElastiCache.

```text
User
 │
 ▼
Application
 │
 ▼
ElastiCache
 │
 ▼
Session Data
 │
 └── Shopping Cart
```

When the user wants to view the shopping cart:

```text
Application
     │
     ▼
ElastiCache
     │
     ▼
Retrieve Session
     │
     ▼
Display Cart
```

---

# 🏗️ ElastiCache with Auto Scaling

Consider an application running on multiple EC2 instances inside an:

```text
Auto Scaling Group
```

The architecture may look like:

```text
                    Users
                      │
                      ▼
                Load Balancer
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
        EC2-1       EC2-2       EC2-3
          │           │           │
          └───────────┼───────────┘
                      │
                      ▼
                 ElastiCache
                      │
                      ▼
                 Session Data
```

The Auto Scaling Group provides:

```text
High Availability

Scalability
```

for the application servers.

---

# ⚠️ The Problem with Local Session Data

Suppose the user's shopping cart is stored directly on:

```text
EC2-1
```

Conceptually:

```text
User
 │
 ▼
Load Balancer
 │
 ▼
EC2-1
 │
 ▼
Shopping Cart Data
```

Now imagine EC2-1 fails.

```text
EC2-1 ❌
   │
   ▼
Instance Replaced
```

Any session information stored only on that instance may also be lost.

---

# ⚡ Using ElastiCache for Session State

Instead of keeping the session information only on the EC2 instance:

```text
EC2
 │
 ▼
Local Session Data
```

the application can keep it in ElastiCache.

```text
                    ElastiCache
                         │
                         ▼
                    Session Data
                         ▲
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
        EC2-1          EC2-2          EC2-3
```

Now the session state is separate from an individual application server.

---

# 🔄 Session State Across Instances

Suppose a user initially connects to:

```text
EC2-1
```

and adds products to the shopping cart.

```text
User
 │
 ▼
EC2-1
 │
 ▼
ElastiCache
 │
 ▼
Shopping Cart
```

If the user's request later reaches another instance:

```text
User
 │
 ▼
EC2-2
 │
 ▼
ElastiCache
 │
 ▼
Same Shopping Cart
```

the application can retrieve the session information from the shared cache.

---

# 🛡️ Auto Scaling Failure Scenario

Consider:

```text
                    Load Balancer
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
           EC2-1        EC2-2        EC2-3
             │
             └───────────┼────────────┘
                         ▼
                    ElastiCache
```

If:

```text
EC2-1 ❌
```

the Auto Scaling Group can replace it.

Because session information is stored separately in ElastiCache:

```text
New EC2 Instance
       │
       ▼
ElastiCache
       │
       ▼
Session Data
```

the application does not depend on session information being stored only on the failed instance.

---

# 🧠 Two Important ElastiCache Use Cases

The lesson focuses on two major use cases.

## 1. Database Query Caching

```text
Application
     │
     ▼
ElastiCache
     │
     ├── Cache Hit ─────► Return Data
     │
     └── Cache Miss
             │
             ▼
          Database
```

Purpose:

```text
Reduce Repeated
Database Queries
```

---

## 2. Session State

```text
Application Servers
        │
        ▼
   ElastiCache
        │
        ▼
   Session Data
```

Purpose:

```text
Store Temporary
Application Session Data
```

---

# 🆚 Database vs ElastiCache

| Area              | Database                           | ElastiCache                                 |
| ----------------- | ---------------------------------- | ------------------------------------------- |
| Primary Purpose   | Store application data             | Cache frequently accessed or temporary data |
| Query Speed       | Requires database processing       | In-memory access                            |
| Repeated Queries  | Database processes them repeatedly | Cached result can be reused                 |
| Session Data      | Can be stored                      | Suitable use case described in lesson       |
| Storage Type      | Backend database storage           | In-memory data store                        |
| Application Logic | Direct queries                     | Application must understand cache behavior  |

The important point is:

```text
ElastiCache
     ≠
Replacement for Database
```

Instead:

```text
ElastiCache
     +
Database
     =
Improved Application Architecture
```

---

# 🏗️ Complete Application Architecture

A complete architecture based on the concepts introduced in the lesson could look like:

```text
                        Users
                          │
                          ▼
                    Load Balancer
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
           EC2-1        EC2-2        EC2-3
             │            │            │
             └────────────┼────────────┘
                          │
                          ▼
                     ElastiCache
                          │
                    ┌─────┴─────┐
                    │           │
                    ▼           ▼
                Cache Hit   Cache Miss
                    │           │
                    │           ▼
                    │        Database
                    │           │
                    │           ▼
                    │      Update Cache
                    │           │
                    └─────┬─────┘
                          ▼
                     Application
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking ElastiCache Replaces the Database

ElastiCache is used alongside the database.

```text
Application
     │
     ▼
ElastiCache
     │
     ▼
Database
```

The backend database still remains the primary data store in the architecture discussed in this lesson.

---

## Mistake 2: Thinking the Application Does Not Need Cache Logic

The application must understand how to interact with the cache.

The application needs logic such as:

```text
Check Cache

If Hit
    Return Cached Data

If Miss
    Query Database
    Update Cache
    Return Data
```

---

## Mistake 3: Querying the Database Before Checking the Cache

For cached data, the architecture introduced in the lesson checks:

```text
Cache First
```

before making an unnecessary database query.

---

## Mistake 4: Forgetting to Populate the Cache After a Miss

A cache miss should lead to:

```text
Cache Miss
    │
    ▼
Query Database
    │
    ▼
Retrieve Data
    │
    ▼
Update Cache
```

Otherwise, future requests may continue producing cache misses.

---

## Mistake 5: Storing Session State Only on EC2

In a scalable architecture:

```text
EC2 Instance
```

may fail or be replaced.

Keeping session state only on an individual EC2 instance can therefore create problems.

The lesson introduces ElastiCache as a shared location for session state.

---

# ✅ Best Practices

* Consider caching when applications repeatedly request the same information.
* Use ElastiCache to reduce unnecessary queries to backend databases.
* Consider ElastiCache for read-heavy workloads.
* Design application logic to check the cache before querying the database.
* When a cache miss occurs, retrieve the data from the database and populate the cache.
* Consider ElastiCache for temporary application data such as session state.
* Avoid depending on individual EC2 instances to maintain session state in an Auto Scaling architecture.
* Use shared session storage when application requests can be handled by different instances.
* Understand that adding caching requires application awareness and appropriate application logic.

---

# ❓ Interview Questions

### Q1. What is Amazon ElastiCache?

Amazon ElastiCache is an in-memory data store that applications can use to cache frequently accessed or temporary information.

---

### Q2. Why would I use ElastiCache?

To reduce repeated database queries and provide faster access to frequently requested information.

---

### Q3. What type of workload is ElastiCache useful for?

The lesson specifically identifies:

```text
Read-Heavy Workloads
```

as a strong use case.

---

### Q4. How does ElastiCache store information?

The lesson describes it using:

```text
Key → Value
```

storage.

---

### Q5. What engines are mentioned for ElastiCache?

The lesson mentions:

```text
Valkey

Memcached

Redis OSS
```

---

### Q6. What is a cache miss?

A cache miss occurs when the application requests information that is not currently available in the cache.

---

### Q7. What should happen after a cache miss?

```text
Cache Miss
    │
    ▼
Query Database
    │
    ▼
Retrieve Data
    │
    ▼
Update Cache
    │
    ▼
Return Data
```

---

### Q8. What is a cache hit?

A cache hit occurs when the requested information already exists in the cache.

The application can return the cached information without querying the backend database.

---

### Q9. Why is a cache hit useful?

It avoids an unnecessary database query and provides faster access to the requested information.

---

### Q10. Does the application need to know about ElastiCache?

Yes.

The lesson emphasizes that application logic must be designed to interact with the cache appropriately.

---

### Q11. Why is caching valuable when thousands of users request the same content?

Without caching, the database may repeatedly process similar queries.

With caching:

```text
Many Users
    │
    ▼
ElastiCache
    │
    ▼
Cached Result
```

the number of backend database queries can be reduced.

---

### Q12. What is another major ElastiCache use case besides database query caching?

```text
Session Data
```

---

### Q13. What is an example of session data?

The lesson uses an e-commerce:

```text
Shopping Cart
```

as an example.

---

### Q14. Why might storing session state directly on an EC2 instance be a problem?

If the EC2 instance fails or is replaced, information stored only on that instance may be lost.

---

### Q15. How does ElastiCache help with Auto Scaling applications?

Multiple application instances can use ElastiCache as a shared location for session information.

```text
EC2-1 ───┐
EC2-2 ───┼────► ElastiCache
EC2-3 ───┘
```

---

### Q16. Does ElastiCache eliminate the need for a backend database?

No.

In the architecture discussed in the lesson, ElastiCache works alongside the backend database.

---

### Q17. What is the basic caching workflow?

```text
Request
   │
   ▼
Check Cache
   │
   ├── HIT ─────► Return Data
   │
   └── MISS
         │
         ▼
      Database
         │
         ▼
     Update Cache
         │
         ▼
     Return Data
```

---

### Q18. What should I think about when I see a requirement to reduce repeated database queries?

```text
Amazon ElastiCache
```

is one of the solutions introduced in this lesson.

---

### Q19. What should I think about when application servers need shared temporary session state?

The lesson introduces:

```text
Amazon ElastiCache
```

for this type of use case.

---

### Q20. What is the easiest way to remember ElastiCache?

```text
Repeated Reads
     │
     ▼
Check Cache
     │
 ┌───┴───┐
 │       │
HIT     MISS
 │       │
 ▼       ▼
Fast   Database
Return    │
          ▼
      Update Cache
```

---

# 💡 Key Takeaways

* Amazon ElastiCache is an in-memory data store.
* It can store data using a key-value model.
* ElastiCache is useful for read-heavy workloads.
* Caching can reduce repeated queries to backend databases.
* Fewer repeated database queries can reduce database resource consumption.
* Cached information can provide faster and lower-latency access.
* The application must be designed to understand and interact with the cache.
* A cache miss occurs when requested information is not available in the cache.
* After a cache miss, the application queries the database and can populate the cache.
* A cache hit occurs when the requested information already exists in the cache.
* A cache hit allows the application to avoid another database query.
* The lesson mentions Valkey, Memcached, and Redis OSS as ElastiCache engines.
* ElastiCache can also store temporary information such as session data.
* Shopping cart information is an example of session state.
* Shared session storage is useful when applications run across multiple EC2 instances.
* Keeping session information outside individual EC2 instances helps support scalable application architectures.

The simplest mental model is:

```text
FIRST REQUEST

User
 │
 ▼
Application
 │
 ▼
Cache
 │
 ▼
MISS ❌
 │
 ▼
Database
 │
 ▼
Get Data
 │
 ▼
Update Cache
 │
 ▼
Return Data
```

Then:

```text
NEXT REQUEST

User
 │
 ▼
Application
 │
 ▼
Cache
 │
 ▼
HIT ✅
 │
 ▼
Return Data

No Database Query Needed
```

And for session state:

```text
                  Load Balancer
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
        EC2-1         EC2-2         EC2-3
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                   ElastiCache
                        │
                        ▼
                  Session State
                        │
                        ▼
                  Shopping Cart
```

---

# 📚 Related Topics

* Amazon ElastiCache
* In-Memory Caching
* Valkey
* Memcached
* Redis OSS
* Cache Hit
* Cache Miss
* Key-Value Stores
* Amazon RDS
* Amazon Aurora
* Session State
* Amazon EC2
* Elastic Load Balancing
* EC2 Auto Scaling
