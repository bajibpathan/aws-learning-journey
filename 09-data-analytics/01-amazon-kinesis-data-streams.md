# Amazon Kinesis Data Streams – Fundamentals and Architecture

> **Amazon Kinesis Data Streams (KDS)** is a scalable, durable, real-time data streaming service that enables applications to continuously collect, process, and analyze streaming data from multiple sources.

---

# 1. Overview

Businesses generate enormous amounts of data every day.

Examples include:

- Website activity
- Customer interactions
- Application logs
- Financial transactions
- Database events
- Social media activity
- IoT device telemetry
- Location tracking information

Organizations analyze this information to understand customer behavior, identify trends, improve products, detect problems, and make business decisions.

AWS provides several managed analytics services to collect, process, store, and analyze data.

One of these services is **Amazon Kinesis Data Streams**.

In this lesson, we will cover:

1. What is Amazon Kinesis?
2. What is Kinesis Data Streams?
3. Real-time data streaming
4. Kinesis architecture
5. Producers and consumers
6. Streams, shards, and records
7. Partition keys and sequence numbers
8. Shard throughput
9. Data retention
10. Kinesis Producer Library
11. Kinesis Client Library
12. Resharding
13. On-demand and provisioned capacity
14. Kinesis vs Amazon SQS
15. Real-world scenarios
16. AWS certification questions

---

# 2. What Is Amazon Kinesis?

Amazon Kinesis is a family of AWS services designed to collect and process streaming data.

Streaming data is information that arrives continuously rather than being collected and processed only in large batches.

### Examples

Consider an e-commerce website.

Customers continuously:

- Browse products
- Search for items
- Add products to shopping carts
- Complete purchases
- Submit reviews

Each interaction generates data.

Instead of waiting until the end of the day to analyze this information, an organization can process it continuously.

### Traditional Batch Processing

```text
Customer Activities
        |
        v
  Collect Data
        |
        v
   Store Data
        |
        v
 Wait for Batch Job
        |
        v
   Analyze Data
```

### Streaming Data Processing

```text
Customer Activities
        |
        v
 Continuous Events
        |
        v
 Amazon Kinesis
        |
        v
 Streaming Consumers
        |
        v
 Real-Time Analysis
```

### Key Concept

**Streaming analytics enables organizations to analyze data as it arrives.**

---

# 3. What Is Amazon Kinesis Data Streams?

Amazon Kinesis Data Streams is a managed service that continuously collects streaming data.

It supports applications that need to process large volumes of incoming data with low latency.

### Common Data Sources

| Data Source | Example |
|---|---|
| Website Clickstreams | Pages viewed by customers |
| Application Logs | Errors and application activity |
| Financial Transactions | Payment and trading events |
| Database Events | Changes to database records |
| Social Media | User interactions and activity |
| IoT Devices | Sensor readings |
| Location Tracking | GPS updates |
| Customer Interactions | Product searches and purchases |

### Example Architecture

```text
                  DATA SOURCES
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   Website Logs    IoT Devices    Transactions
        |              |              |
        +--------------+--------------+
                       |
                       v
                Data Producers
                       |
                       v
             Kinesis Data Streams
                       |
                       v
                Data Consumers
                       |
            +----------+----------+
            |          |          |
            v          v          v
         Analytics   Alerts     Storage
```

### Important Concept

Kinesis Data Streams is designed for **real-time streaming data ingestion and processing**.

---

# 4. Real-Time vs Near-Real-Time Processing

The lecture emphasizes an important distinction between Kinesis Data Streams and Amazon Data Firehose.

### Real-Time Processing

Data is made available for consumption with low latency.

**Example:** Detecting unusual financial transactions shortly after they occur.

### Near-Real-Time Processing

Data may be buffered or grouped before being delivered to its destination.

**Example:** Collecting application logs and delivering them periodically to Amazon S3.

### Comparison

| Feature | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| Primary Purpose | Real-time streaming | Managed streaming data delivery |
| Processing Model | Consumer applications read records | Service delivers records to destinations |
| Latency | Low-latency streaming | Often near-real-time due to buffering |
| Custom Consumers | Supported | Not required for basic delivery |
| Typical Use Case | Real-time analytics | Deliver streaming data to storage and analytics destinations |

**AWS terminology note:** The service previously called *Kinesis Data Firehose* is now called *Amazon Data Firehose*.

### AWS Exam Point

**Kinesis Data Streams → Real-time streaming**

**Amazon Data Firehose → Managed data delivery, commonly near-real-time**

---

# 5. Kinesis Data Streams Architecture

Kinesis Data Streams uses a producer-consumer architecture.

### Main Components

1. Producers
2. Kinesis data stream
3. Shards
4. Data records
5. Consumers

### Architecture

```text
                      PRODUCERS
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Web App      IoT Device    Log Agent
             |            |            |
             +------------+------------+
                          |
                          v
                 Kinesis Data Stream
                          |
             +------------+------------+
             |            |            |
             v            v            v
           Shard 1      Shard 2      Shard 3
             |            |            |
             +------------+------------+
                          |
                          v
                      CONSUMERS
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Analytics     Monitoring     Storage
```

### How It Works

1. Applications generate streaming data.
2. Producers send records to Kinesis Data Streams.
3. Kinesis distributes records across shards using partition keys.
4. Consumers read records from the stream.
5. Consumers process the information.
6. Processed data can be stored or sent to other services.

---

# 6. Understanding Producers

A **producer** is an application or service that sends data into a Kinesis data stream.

### Examples

- Web applications
- Mobile applications
- IoT devices
- Application servers
- Logging agents
- Financial transaction systems

### Architecture

```text
                  Application
                       |
                       v
                 Generate Event
                       |
                       v
                    Producer
                       |
                       v
                Kinesis Data Stream
```

### Example

A customer visits an online shopping website.

The application generates an event:

```json
{
  "customerId": "CUST1001",
  "eventType": "product_view",
  "productId": "PROD5001",
  "timestamp": "2026-11-02T10:30:00Z"
}
```

A producer application sends this event into Kinesis Data Streams.

### Important Concept

Producers are responsible for writing records into the stream.

---

# 7. Understanding Consumers

A **consumer** is an application or service that reads records from a Kinesis data stream.

Consumers process records according to application requirements.

### Examples

- Real-time analytics applications
- Fraud detection systems
- Monitoring applications
- Data processing applications
- Applications that persist records to storage

### Architecture

```text
                Kinesis Data Stream
                         |
                         v
                      Consumer
                         |
                         v
                   Process Records
                         |
              +----------+----------+
              |          |          |
              v          v          v
           Generate    Update      Store
            Alerts    Dashboard    Results
```

### Example

A financial application continuously receives transaction records.

A consumer analyzes those transactions and identifies suspicious activity.

If suspicious activity is detected, the application generates an alert.

### Important Concept

**Producers write records; consumers read and process records.**

---

# 8. What Is a Kinesis Data Stream?

A Kinesis data stream is a collection of shards.

Each shard contains an ordered sequence of data records.

### Architecture

```text
                  Kinesis Data Stream
                          |
             +------------+------------+
             |            |            |
             v            v            v
           Shard 1      Shard 2      Shard 3
             |            |            |
             v            v            v
           Record A     Record D     Record G
           Record B     Record E     Record H
           Record C     Record F     Record I
```

### Important Concept

A stream consists of one or more shards.

The number of shards determines the stream's capacity in provisioned mode.

---

# 9. What Is a Shard?

A **shard** is a uniquely identified sequence of data records within a Kinesis data stream.

Shards provide the underlying capacity for reading and writing streaming records.

### Architecture

```text
                    Kinesis Stream
                          |
                          v
                        Shard
                          |
               +----------+----------+
               |          |          |
               v          v          v
             Record 1   Record 2   Record 3
```

Each shard contains records in a sequence.

### Shard Characteristics

- Stores an ordered sequence of records
- Has a unique shard identifier
- Provides defined read and write capacity in provisioned mode
- Can be split to increase capacity
- Can be merged to reduce capacity

### Important Concept

**Shards are the fundamental capacity units of Kinesis Data Streams in provisioned mode.**

---

# 10. Shard Throughput

In provisioned capacity mode, each shard provides a defined amount of throughput.

### Per-Shard Capacity

| Operation | Capacity |
|---|---|
| Write Throughput | 1 MiB/second |
| Write Records | 1,000 records/second |
| Shared Read Throughput | 2 MiB/second |
| Classic `GetRecords` API Calls | Up to 5 calls/second per shard |

### Example

Suppose a stream contains three shards.

The aggregate provisioned capacity is approximately:

| Metric | Per Shard | Three Shards |
|---|---:|---:|
| Write Throughput | 1 MiB/s | 3 MiB/s |
| Write Records | 1,000/s | 3,000/s |
| Shared Read Throughput | 2 MiB/s | 6 MiB/s |

### Important Clarification

The lecture uses the terms *megabits* and *megabytes* interchangeably.

For AWS capacity calculations, the important values are:

**1 MiB/second write capacity per shard**

**2 MiB/second shared read capacity per shard**

These are byte-based throughput measurements, not megabits per second.

The five-read-calls-per-second limit applies to the classic `GetRecords` polling API, not every possible consumer configuration.

---

# 11. What Is a Data Record?

A **data record** is the unit of information stored in Kinesis Data Streams.

The lecture identifies three important components:

1. Partition key
2. Sequence number
3. Data blob

### Architecture

```text
                    Kinesis Record
                          |
             +------------+------------+
             |            |            |
             v            v            v
         Partition     Sequence      Data Blob
            Key         Number
```

### Components

| Component | Description |
|---|---|
| Partition Key | Determines shard placement |
| Sequence Number | Identifies a record's position within its shard |
| Data Blob | Contains the actual record data |

### Example

```text
Partition Key: customer-1001

Sequence Number: 496600000000000000001

Data Blob:
{
  "event": "purchase",
  "amount": 250
}
```

### Important Concept

Kinesis Data Streams stores the record data without interpreting the application-specific contents.

**Technical clarification:** The lecture describes the data blob as mutable. Once a record is written to Kinesis Data Streams, its stored data is not updated in place.

---

# 12. Understanding Partition Keys

A partition key determines which shard receives a record.

The producer provides the partition key when writing a record.

Kinesis uses the partition key to determine shard placement.

### Example

```text
                      Producer
                         |
                         v
                  Partition Keys
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
         Customer A   Customer B   Customer C
             |           |           |
             v           v           v
           Shard 1     Shard 2     Shard 3
```

This diagram illustrates possible distribution. Actual shard placement depends on the partition key's hash value and shard hash-key ranges.

### Why Are Partition Keys Important?

Partition keys influence:

- Record distribution
- Shard utilization
- Data ordering
- Streaming throughput

### Example

Suppose an application sends customer events.

```text
customer-1001
customer-1002
customer-1003
```

Kinesis uses each partition key to determine where the records are stored.

### Important Concept

**Records with the same partition key are routed to the same active shard at a given point in time.**

If shards are split or merged, subsequent records may be routed to a different shard.

---

# 13. Partition Keys and Record Ordering

Kinesis maintains record ordering within a shard.

### Example

```text
                    Shard 1
                       |
                       v
                 Order Created
                       |
                       v
                 Order Updated
                       |
                       v
                 Order Completed
```

These records can be processed in shard sequence.

### Important Limitation

Kinesis does not guarantee a single global ordering across all shards.

### AWS Exam Point

**Ordering is maintained within a shard, not across the entire multi-shard stream.**

For applications that require ordering of related records, choose partition keys carefully.

---

# 14. Data Retention

Kinesis Data Streams retains records for a configurable period.

### Retention Period

| Configuration | Duration |
|---|---|
| Default | 24 hours |
| Maximum | 365 days |

### Why Is Retention Important?

Retention allows consumers to read records that were previously written to the stream.

It is useful when:

- A consumer temporarily stops
- An application needs to recover from a failure
- Historical records need to be reprocessed
- Multiple consumers process records independently

### Example

```text
                  Producer
                     |
                     v
             Kinesis Data Stream
                     |
                     v
               Stored Records
                     |
                     v
                 Consumer
```

If the consumer becomes unavailable temporarily, records remain available until their retention period expires.

### Important Concept

Unlike a typical SQS processing workflow, reading a Kinesis record does not remove it from the stream.

---

# 15. Kinesis Producer Library (KPL)

The **Kinesis Producer Library (KPL)** helps applications efficiently send records to Kinesis Data Streams.

### Purpose

KPL helps optimize producer-side data ingestion.

### Benefits

- Efficient record publishing
- Batching
- Record aggregation
- Improved throughput utilization

### Architecture

```text
                  Application
                       |
                       v
             Kinesis Producer Library
                       |
                       v
                Kinesis Data Streams
```

### Example

A website generates thousands of clickstream events.

Rather than sending each event inefficiently, a producer can use KPL capabilities to optimize record transmission.

### Important Concept

**KPL optimizes producer applications.**

---

# 16. Kinesis Client Library (KCL)

The **Kinesis Client Library (KCL)** helps developers build applications that consume records from Kinesis Data Streams.

### Purpose

KCL simplifies consumer-side stream processing.

### Benefits

- Coordinates record processing
- Supports multiple workers
- Manages shard assignments
- Supports checkpointing
- Helps recover from worker failures

### Architecture

```text
                Kinesis Data Streams
                         |
                         v
               Kinesis Client Library
                         |
                         v
                 Consumer Workers
                         |
              +----------+----------+
              |          |          |
              v          v          v
           Worker 1   Worker 2   Worker 3
```

### Important Concept

**KCL helps manage consumer applications.**

### KPL vs KCL

| Feature | KPL | KCL |
|---|---|---|
| Full Name | Kinesis Producer Library | Kinesis Client Library |
| Purpose | Write records | Read and process records |
| Used By | Producers | Consumers |
| Main Benefit | Efficient ingestion | Coordinated processing |

---

# 17. What Is Resharding?

Resharding is the process of changing the number of shards in a Kinesis data stream.

In provisioned mode, this allows you to adjust stream capacity.

Two important operations are:

1. Split shards
2. Merge shards

---

# 18. Splitting Shards

Splitting divides one shard into two child shards.

### Before Splitting

```text
                 Kinesis Stream
                       |
                       v
                     Shard 1
```

### After Splitting

```text
                 Kinesis Stream
                       |
              +--------+--------+
              |                 |
              v                 v
            Shard 1A          Shard 1B
```

### Why Split Shards?

Splitting increases available shard capacity.

### Example

A streaming application experiences increased traffic.

The existing shard capacity is insufficient.

The stream can be resharded to increase capacity.

### Important Concept

**Splitting shards increases provisioned capacity and can increase cost.**

---

# 19. Merging Shards

Merging combines two adjacent shards into one shard.

### Before Merging

```text
                 Kinesis Stream
                       |
              +--------+--------+
              |                 |
              v                 v
            Shard 1           Shard 2
```

### After Merging

```text
                 Kinesis Stream
                       |
                       v
                  Merged Shard
```

### Why Merge Shards?

Merging can reduce unused provisioned capacity.

### Example

An application experiences lower traffic during off-peak hours.

The organization may reduce provisioned shard capacity to optimize costs.

### Important Concept

**Merging shards reduces provisioned capacity and may reduce cost.**

---

# 20. Kinesis Capacity Modes

The lecture introduces two capacity modes:

1. On-demand
2. Provisioned

---

# 21. On-Demand Capacity Mode

In on-demand mode, Kinesis automatically manages stream capacity.

You do not manually specify the number of shards.

### Architecture

```text
                 Streaming Producers
                         |
                         v
                  Incoming Records
                         |
                         v
                Kinesis On-Demand
                         |
                         v
              Automatically Managed
                     Capacity
                         |
                         v
                      Consumers
```

### Benefits

- Less capacity planning
- Automatic capacity management
- Suitable for variable traffic
- Reduced operational overhead

### Common Use Cases

- Unpredictable traffic
- Applications with changing ingestion rates
- Streaming workloads requiring simplified capacity management

### Capacity Clarification

The lecture describes an initial capacity of approximately 4 MiB/second or 4,000 records/second.

These figures are associated with the historical on-demand capacity model. AWS has introduced additional on-demand scaling capabilities and options, so the exact applicable throughput depends on the stream's current configuration and service limits.

For certification preparation, focus on the architectural distinction:

**On-demand mode automatically manages capacity.**

---

# 22. Provisioned Capacity Mode

In provisioned mode, you specify the number of shards.

Each shard provides a defined amount of read and write throughput.

### Architecture

```text
                    Producers
                        |
                        v
                Kinesis Data Stream
                        |
             +----------+----------+
             |          |          |
             v          v          v
           Shard 1    Shard 2    Shard 3
             |          |          |
             +----------+----------+
                        |
                        v
                     Consumers
```

### Example

A company provisions five shards.

The aggregate write capacity is approximately:

```text
5 shards × 1 MiB/second

= 5 MiB/second
```

The aggregate shared read capacity is approximately:

```text
5 shards × 2 MiB/second

= 10 MiB/second
```

These calculations assume the workload is distributed appropriately across the shards.

### Benefits

- Explicit capacity control
- Predictable provisioned throughput
- Ability to adjust shard count
- Suitable for workloads with known capacity requirements

### Important Concept

**Provisioned mode requires capacity planning and shard management.**

---

# 23. On-Demand vs Provisioned

| Feature | On-Demand | Provisioned |
|---|---|---|
| Capacity Management | Automatic | User-managed |
| Shard Count | Managed by AWS | Configured by user |
| Scaling | Automatic | Manual or automated shard scaling |
| Pricing Basis | Usage-based capacity model | Provisioned shard capacity and applicable usage |
| Operational Effort | Lower | Higher |
| Suitable Workloads | Variable or unpredictable | Predictable workloads |
| Capacity Planning | Less required | Required |

### AWS Exam Point

Choose **on-demand** when traffic is unpredictable and you want AWS to manage stream capacity.

Choose **provisioned** when you need explicit control over shard capacity.

---

# 24. Kinesis Data Streams vs Amazon SQS

This comparison is particularly important for AWS certification scenarios.

Both services can receive data from producers and make it available to consumers.

However, they are designed for different purposes.

### Amazon SQS

```text
Producer
   |
   v
SQS Queue
   |
   v
Consumer
   |
   v
Process Message
   |
   v
Delete Message
```

### Kinesis Data Streams

```text
Producer
   |
   v
Kinesis Stream
   |
   +----------+----------+
   |          |          |
   v          v          v
Consumer A Consumer B Consumer C
   |          |          |
   v          v          v
Analytics   Alerts     Storage
```

### Comparison

| Feature | Amazon SQS | Kinesis Data Streams |
|---|---|---|
| Primary Purpose | Message queuing | Real-time data streaming |
| Common Use Case | Application decoupling | Streaming analytics |
| Consumer Model | Consumers retrieve messages | Consumers read stream records |
| Record Retention | 4 days default | 24 hours default |
| Maximum Retention | 14 days | 365 days |
| Message Deletion | Consumer deletes after processing | Reading does not delete records |
| Replay | Limited by queue processing behavior | Supported within retention period |
| Multiple Independent Consumers | Requires appropriate queue architecture | Supported |
| Ordering | FIFO queues provide ordering guarantees | Ordering within shards |
| Scaling Model | Managed queue scaling | On-demand or provisioned capacity |

### Important Concept

**SQS is designed for message queuing and application decoupling.**

**Kinesis Data Streams is designed for continuous streaming data ingestion and processing.**

---

# 25. Real-World Example: E-Commerce Clickstream Analytics

Consider an e-commerce platform.

Customers continuously interact with the website.

### Events

- Product viewed
- Product searched
- Item added to cart
- Checkout initiated
- Purchase completed

The business wants to analyze these activities in real time.

### Architecture

```text
                   E-Commerce Website
                           |
                           v
                     User Activities
                           |
                           v
                     Event Producer
                           |
                           v
                  Kinesis Data Streams
                           |
               +-----------+-----------+
               |           |           |
               v           v           v
          Analytics     Monitoring   Data Storage
               |           |           |
               v           v           v
          Dashboards      Alerts     Historical Data
```

### Benefits

- Understand customer behavior
- Monitor user interactions
- Identify popular products
- Generate operational alerts
- Support real-time business decisions

---

# 26. Real-World Example: Application Log Streaming

Organizations generate large volumes of application logs.

These logs may come from:

- Web servers
- Application servers
- Containers
- Distributed services

### Architecture

```text
                    Application Servers
                            |
                +-----------+-----------+
                |           |           |
                v           v           v
              App 1       App 2       App 3
                |           |           |
                +-----------+-----------+
                            |
                            v
                       Log Producer
                            |
                            v
                    Kinesis Data Streams
                            |
                            v
                      Log Consumers
                            |
                +-----------+-----------+
                |                       |
                v                       v
          Real-Time Alerts        Persistent Storage
```

### Benefits

- Centralized log ingestion
- Real-time event processing
- Faster anomaly detection
- Integration with analytics systems

---

# 27. Real-World Example: Financial Transaction Monitoring

A financial institution processes large volumes of transactions.

The organization wants to identify suspicious transactions quickly.

### Architecture

```text
                  Financial Transactions
                            |
                            v
                     Transaction Events
                            |
                            v
                    Kinesis Data Streams
                            |
                            v
                   Fraud Detection Consumer
                            |
                            v
                     Analyze Transactions
                            |
                  +---------+---------+
                  |                   |
                  v                   v
              Normal Event      Suspicious Event
                  |                   |
                  v                   v
             Continue Flow       Generate Alert
```

### Benefits

- Low-latency transaction analysis
- Continuous monitoring
- Faster fraud detection
- Scalable streaming processing

---

# 28. Common Mistakes

### Mistake 1: Confusing Kinesis with SQS

Kinesis Data Streams is designed for streaming data processing.

SQS is designed for message queuing.

### Mistake 2: Assuming Kinesis Records Are Deleted After Consumption

Reading a record does not remove it from the stream.

Records remain available according to the configured retention period.

### Mistake 3: Confusing Megabits and Mebibytes

Shard throughput is expressed using byte-based measurements.

### Mistake 4: Assuming Global Ordering Across Shards

Ordering is maintained within a shard, not across the entire stream.

### Mistake 5: Using Poor Partition Keys

A poorly chosen partition key can concentrate traffic on a single shard.

This can create a hot-shard problem.

### Mistake 6: Forgetting Retention Limits

Kinesis Data Streams retains records for a limited period.

It is not a replacement for permanent storage.

### Mistake 7: Assuming Provisioned Mode Automatically Manages Every Capacity Requirement

Provisioned mode requires shard capacity management.

### Mistake 8: Confusing KPL and KCL

KPL is used by producers.

KCL is used by consumers.

---

# 29. AWS Certification Exam Scenarios

## Scenario 1: Real-Time Clickstream Processing

An e-commerce company wants to continuously capture and analyze customer interactions with minimal delay.

**Question:** Which AWS service is most suitable?

**Answer:** Amazon Kinesis Data Streams.

**Reason:** Kinesis Data Streams supports low-latency streaming ingestion and processing.

---

## Scenario 2: Application Decoupling

An application needs to place tasks into a queue so background workers can process them asynchronously.

**Question:** Which AWS service should be used?

**Answer:** Amazon SQS.

**Reason:** SQS is designed for message queuing and application decoupling.

---

## Scenario 3: Streaming Record Retention

A company needs to retain streaming records for 30 days so consumers can replay historical events.

**Question:** Which service supports this requirement?

**Answer:** Amazon Kinesis Data Streams.

**Reason:** Kinesis supports configurable retention up to 365 days.

---

## Scenario 4: Unpredictable Streaming Traffic

A company expects unpredictable changes in incoming streaming traffic.

The operations team does not want to manage shard capacity manually.

**Question:** Which capacity mode should be selected?

**Answer:** On-demand capacity mode.

**Reason:** Kinesis automatically manages capacity.

---

## Scenario 5: Predictable Streaming Traffic

A company knows its required streaming throughput and wants to configure capacity explicitly.

**Question:** Which capacity mode should be used?

**Answer:** Provisioned capacity mode.

**Reason:** Provisioned mode allows explicit shard capacity management.

---

## Scenario 6: Increase Stream Capacity

A stream in provisioned mode is experiencing insufficient throughput.

**Question:** What operation can increase the number of shards?

**Answer:** Split shards.

**Reason:** Splitting increases the number of active shards and available capacity.

---

## Scenario 7: Reduce Unused Capacity

A company has excess provisioned shard capacity during periods of low traffic.

**Question:** What operation can reduce the number of shards?

**Answer:** Merge shards.

**Reason:** Merging reduces provisioned shard capacity.

---

## Scenario 8: Optimize Producer Applications

A company needs to efficiently publish large volumes of streaming records.

**Question:** Which library can help?

**Answer:** Kinesis Producer Library.

---

## Scenario 9: Build Consumer Applications

A company needs to coordinate multiple workers processing records from Kinesis Data Streams.

**Question:** Which library is suitable?

**Answer:** Kinesis Client Library.

---

## Scenario 10: Preserve Related Record Ordering

A company wants related customer events to maintain ordering during stream processing.

**Question:** What should the application consider?

**Answer:** Use appropriate partition keys.

**Reason:** Records are ordered within shards, and partition keys determine shard placement.

---

# 30. Interview Questions

### Q1. What is Amazon Kinesis Data Streams?

A managed service for continuously collecting and processing streaming data with low latency.

### Q2. What are common Kinesis use cases?

Clickstream analytics, log streaming, financial transactions, IoT telemetry, and real-time monitoring.

### Q3. What is a producer?

An application that writes records into a Kinesis data stream.

### Q4. What is a consumer?

An application that reads and processes records from a Kinesis data stream.

### Q5. What is a shard?

A uniquely identified sequence of records that provides stream capacity.

### Q6. What is the write capacity of a provisioned shard?

1 MiB per second or 1,000 records per second.

### Q7. What is the shared read capacity of a provisioned shard?

2 MiB per second.

### Q8. What is a partition key?

A value used to determine which shard receives a record.

### Q9. What is a sequence number?

An identifier assigned to a record within its shard.

### Q10. What is the default Kinesis retention period?

24 hours.

### Q11. What is the maximum retention period?

365 days.

### Q12. What is KPL?

Kinesis Producer Library, which helps optimize producer applications.

### Q13. What is KCL?

Kinesis Client Library, which helps coordinate consumer applications.

### Q14. What is resharding?

Changing the number of shards in a Kinesis data stream.

### Q15. What happens when a shard is split?

One shard becomes two child shards, increasing provisioned capacity.

### Q16. What happens when shards are merged?

Two adjacent shards are combined into one, reducing provisioned capacity.

### Q17. What is on-demand mode?

A capacity mode where AWS automatically manages streaming capacity.

### Q18. What is provisioned mode?

A capacity mode where the user specifies shard capacity.

### Q19. What is the difference between Kinesis and SQS?

Kinesis is designed for streaming data processing, while SQS is designed for message queuing.

### Q20. Does reading a Kinesis record delete it?

No. Records remain available until their retention period expires.

---

# 31. Key Takeaways

- Amazon Kinesis Data Streams is a real-time streaming service.
- It continuously ingests large volumes of streaming data.
- Producers send records into streams.
- Consumers read and process records.
- A Kinesis data stream consists of shards.
- Shards are capacity units in provisioned mode.
- Each provisioned shard supports 1 MiB/second of writes.
- Each provisioned shard supports 2 MiB/second of shared reads.
- Partition keys determine shard placement.
- Records within a shard have sequence numbers.
- Kinesis preserves ordering within shards.
- Default retention is 24 hours.
- Maximum retention is 365 days.
- KPL helps optimize producer applications.
- KCL helps manage consumer applications.
- Resharding changes stream capacity.
- Splitting shards increases provisioned capacity.
- Merging shards reduces provisioned capacity.
- On-demand mode automatically manages capacity.
- Provisioned mode allows explicit shard management.
- Kinesis Data Streams supports real-time analytics.
- Amazon SQS is primarily designed for application decoupling and message queuing.

---

# 32. The Most Important Concepts to Remember

```text
                  AMAZON KINESIS DATA STREAMS
                  ===========================

                         DATA SOURCES
                              |
               +--------------+--------------+
               |              |              |
               v              v              v
           Website Logs    IoT Devices    Transactions
               |              |              |
               +--------------+--------------+
                              |
                              v
                          PRODUCERS
                              |
                              v
                     KINESIS DATA STREAM
                              |
               +--------------+--------------+
               |              |              |
               v              v              v
             SHARD 1        SHARD 2        SHARD 3
               |              |              |
               +--------------+--------------+
                              |
                              v
                          CONSUMERS
                              |
               +--------------+--------------+
               |              |              |
               v              v              v
            Analytics       Alerts         Storage
```

### Exam Memory Points

| Concept | Remember |
|---|---|
| Kinesis Data Streams | Real-time streaming |
| Producer | Writes records |
| Consumer | Reads records |
| Stream | Collection of shards |
| Shard | Capacity and record sequence |
| Partition Key | Determines shard placement |
| Sequence Number | Identifies record position |
| Write Capacity | 1 MiB/s per provisioned shard |
| Shared Read Capacity | 2 MiB/s per provisioned shard |
| Default Retention | 24 hours |
| Maximum Retention | 365 days |
| KPL | Producer optimization |
| KCL | Consumer processing |
| Split Shard | Increase capacity |
| Merge Shards | Decrease capacity |
| On-Demand | Automatically managed capacity |
| Provisioned | User-managed shard capacity |
| SQS | Message queuing |
| Kinesis | Streaming analytics |

---

# 33. Related Topics

- Amazon Kinesis
- Amazon Kinesis Data Streams
- Amazon Data Firehose
- Streaming Data Architecture
- Real-Time Analytics
- Producers and Consumers
- Kinesis Shards
- Partition Keys
- Sequence Numbers
- Kinesis Producer Library
- Kinesis Client Library
- Resharding
- On-Demand Capacity
- Provisioned Capacity
- Amazon SQS
- AWS Data Analytics

**Lesson completed:** Amazon Kinesis Data Streams – Fundamentals
