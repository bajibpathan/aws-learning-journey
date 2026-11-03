# Amazon Data Firehose – Fundamentals and Architecture

> **Amazon Data Firehose** is a fully managed service that captures, transforms, and delivers streaming data to supported destinations, including Amazon S3, Amazon Redshift, Amazon OpenSearch Service, and third-party analytics platforms.

---

# 1. Overview

In the previous lesson, we explored **Amazon Kinesis Data Streams**, a service designed to continuously ingest and process streaming data.

We learned about:

- Streaming data
- Producers and consumers
- Kinesis shards
- Partition keys
- Sequence numbers
- Data retention
- Kinesis Producer Library (KPL)
- Kinesis Client Library (KCL)
- Resharding
- On-demand and provisioned capacity

Kinesis Data Streams enables applications to process data in real time.

However, many organizations also need to store streaming data for long-term analysis.

This is where **Amazon Data Firehose** becomes useful.

In this lesson, we will cover:

1. What is Amazon Data Firehose?
2. Kinesis Data Streams recap
3. Why Data Firehose is needed
4. Data Firehose architecture
5. Data sources
6. Supported destinations
7. Direct ingestion vs Kinesis Data Streams
8. Near-real-time delivery
9. Buffering
10. Data transformation using Lambda
11. Amazon S3 integration
12. Amazon Redshift integration
13. Amazon OpenSearch integration
14. Third-party integrations
15. Kinesis Data Streams vs Data Firehose
16. Real-world use cases
17. AWS certification scenarios
18. Interview questions

---

# 2. Kinesis Data Streams – Quick Recap

Amazon Kinesis Data Streams is designed for real-time streaming data ingestion and processing.

### Architecture

```text
                    DATA SOURCES
                         |
            +------------+------------+
            |            |            |
            v            v            v
        Web Logs      IoT Devices   Transactions
            |            |            |
            +------------+------------+
                         |
                         v
                      PRODUCERS
                         |
                         v
                 KINESIS DATA STREAM
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
         Analytics     Alerts      Storage
```

### Important Characteristics

| Feature | Kinesis Data Streams |
|---|---|
| Primary Purpose | Real-time data streaming |
| Data Ingestion | Continuous |
| Producers | Applications and supported integrations |
| Consumers | Custom applications and supported AWS services |
| Default Retention | 24 hours |
| Maximum Retention | 365 days |
| Record Replay | Supported within retention period |
| Persistent Storage | Requires integration with another service |

### Key Concept

Kinesis Data Streams stores records for a configurable retention period.

It does not automatically archive all incoming records into permanent storage.

Applications or integrated services must deliver those records to a persistent destination.

---

# 3. What Is Amazon Data Firehose?

Amazon Data Firehose is a fully managed streaming data delivery service.

It collects streaming data from supported sources and delivers that data to configured destinations.

### Common Destinations

- Amazon S3
- Amazon Redshift
- Amazon OpenSearch Service
- Splunk
- Supported HTTP endpoints
- Supported third-party analytics platforms

### Architecture

```text
                    DATA SOURCES
                         |
            +------------+------------+
            |            |            |
            v            v            v
        Applications   AWS Logs    Kinesis Streams
            |            |            |
            +------------+------------+
                         |
                         v
                  AMAZON DATA FIREHOSE
                         |
                         v
                 Buffer / Transform
                         |
            +------------+------------+
            |            |            |
            v            v            v
         Amazon S3    OpenSearch    Third-Party
                         |
                         v
                    Data Analysis
```

### Important Concept

**Amazon Data Firehose simplifies the process of delivering streaming data to persistent storage and analytics destinations.**

---

# 4. Why Do We Need Amazon Data Firehose?

Consider an application generating thousands of log events every second.

The organization wants to:

1. Collect the logs continuously.
2. Process or transform them if required.
3. Store them in Amazon S3.
4. Analyze the stored data later.

Without a managed delivery service, developers may need to create and maintain consumer applications.

### Without Data Firehose

```text
Application Logs
       |
       v
Kinesis Data Streams
       |
       v
Custom Consumer Application
       |
       v
Data Processing Logic
       |
       v
Amazon S3
```

The organization is responsible for developing and operating the consumer.

### With Data Firehose

```text
Application Logs
       |
       v
Amazon Data Firehose
       |
       v
Amazon S3
```

Data Firehose manages much of the delivery process.

### Benefits

- Reduced operational overhead
- Managed delivery infrastructure
- Automatic scaling within service limits
- Configurable buffering
- Optional data transformation
- Integration with supported destinations

### Key Concept

Data Firehose eliminates the need to build a custom streaming consumer solely to deliver records to a supported destination.

---

# 5. Amazon Data Firehose Architecture

A typical Data Firehose architecture contains four main components:

1. Data sources
2. Data Firehose delivery stream
3. Optional processing
4. Destination

### Architecture

```text
                      PRODUCERS
                          |
             +------------+------------+
             |            |            |
             v            v            v
         Applications   AWS Logs    Kinesis
             |            |            |
             +------------+------------+
                          |
                          v
                 DATA FIREHOSE STREAM
                          |
                          v
                     BUFFERING
                          |
                          v
                OPTIONAL TRANSFORMATION
                          |
                          v
                     DESTINATION
                          |
             +------------+------------+
             |            |            |
             v            v            v
             S3        OpenSearch    Splunk
```

### Processing Flow

1. A producer generates streaming records.
2. Records are sent to Data Firehose.
3. Data Firehose buffers incoming records.
4. Optional transformation is performed.
5. Data is delivered to the configured destination.
6. The destination stores or processes the records.

---

# 6. Amazon Data Firehose Sources

The lecture identifies several sources that can send data to Amazon Data Firehose.

### Common Sources

| Source | Description |
|---|---|
| Applications | Send records directly |
| Amazon Kinesis Data Streams | Provides streaming records |
| Amazon CloudWatch Logs | Sends log data through a supported integration |
| AWS Services | Supported service integrations |
| Kinesis Agent | Collects and forwards supported data |

### Architecture

```text
                  DATA SOURCES
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Application   CloudWatch    Kinesis
                      Logs      Data Streams
          |            |            |
          +------------+------------+
                       |
                       v
                Amazon Data Firehose
```

### Important Concept

**Kinesis Data Streams is not required to use Amazon Data Firehose.**

Applications can send records directly to Data Firehose.

---

# 7. Direct Ingestion vs Kinesis Data Streams

There are two important architectures to understand.

## Architecture A – Direct Ingestion

Applications send records directly to Data Firehose.

```text
Application
     |
     v
Amazon Data Firehose
     |
     v
Amazon S3
```

### When to Use

Choose direct ingestion when the primary requirement is to deliver streaming data to a supported destination.

### Example

An application generates audit logs that must be stored in Amazon S3.

The application sends records directly to Data Firehose.

---

## Architecture B – Kinesis Data Streams as a Source

Applications first send records to Kinesis Data Streams.

Data Firehose reads records from the stream and delivers them to a destination.

```text
                    Application
                         |
                         v
                Kinesis Data Streams
                         |
              +----------+----------+
              |                     |
              v                     v
       Analytics Consumer      Amazon Data Firehose
              |                     |
              v                     v
       Real-Time Analysis        Amazon S3
```

### When to Use

Choose this architecture when applications require both:

- Real-time stream processing
- Managed delivery to persistent storage

### Example

A financial application needs to analyze transactions in real time while archiving the transaction events in Amazon S3.

### Key Concept

**Kinesis Data Streams handles streaming and replay, while Data Firehose handles managed delivery to supported destinations.**

---

# 8. Real-Time vs Near-Real-Time Delivery

This is one of the most important distinctions in the lecture.

## Kinesis Data Streams

Kinesis Data Streams makes incoming records available to consumers with low latency.

This supports real-time processing applications.

## Amazon Data Firehose

Data Firehose commonly buffers records before delivering them to destinations.

As a result, delivery is often near-real-time rather than immediate.

### Architecture

```text
KINESIS DATA STREAMS
====================

Producer
   |
   v
Kinesis Stream
   |
   v
Consumer
   |
   v
Low-Latency Processing


AMAZON DATA FIREHOSE
====================

Producer
   |
   v
Data Firehose
   |
   v
Buffer Records
   |
   v
Deliver to Destination
```

### Comparison

| Feature | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| Primary Purpose | Real-time streaming | Managed streaming delivery |
| Consumer Processing | Low latency | Delivery often involves buffering |
| Buffering | Records retained in stream | Delivery buffering |
| Custom Consumer | Common | Not required for basic delivery |
| Destination Delivery | Consumer or integration handles it | Managed by Firehose |

### AWS Exam Point

**Kinesis Data Streams = Real-time streaming**

**Amazon Data Firehose = Managed delivery, commonly near-real-time**

---

# 9. Understanding Data Firehose Buffering

Amazon Data Firehose can buffer incoming records before delivering them to the destination.

Buffering helps improve delivery efficiency.

### Architecture

```text
                    Streaming Records
                           |
                           v
                    Amazon Data Firehose
                           |
                           v
                      Buffer Records
                           |
                 +---------+---------+
                 |                   |
                 v                   v
            Size Threshold      Time Threshold
                 |                   |
                 +---------+---------+
                           |
                           v
                    Deliver Records
                           |
                           v
                       Amazon S3
```

### How Buffering Works

For destinations using size-based and interval-based buffering, delivery can occur when either configured threshold is reached.

For example:

| Setting | Example |
|---|---|
| Buffer Size | 1 MiB |
| Buffer Interval | 60 seconds |

### Scenario A – Buffer Size Reached

Suppose incoming records reach 1 MiB after 20 seconds.

Data Firehose can deliver the buffered records without waiting for the full 60 seconds.

### Scenario B – Buffer Interval Reached

Suppose only 500 KiB arrives during 60 seconds.

Data Firehose can deliver the available records when the buffer interval expires.

### Important Clarification

The lecture uses **1 MB or 60 seconds** to explain buffering.

These are illustrative configuration values, not universal fixed requirements.

Actual buffering options depend on the destination and delivery configuration.

Some destinations support different buffering intervals, and certain delivery modes support lower-latency delivery.

### Important Concept

**Buffering balances delivery efficiency and latency.**

---

# 10. Data Transformation Using AWS Lambda

Amazon Data Firehose supports optional data transformation using AWS Lambda.

This allows organizations to modify incoming records before delivering them to their destination.

### Architecture

```text
                    Streaming Data
                          |
                          v
                  Amazon Data Firehose
                          |
                          v
                    AWS Lambda
                          |
                          v
                   Transform Records
                          |
                          v
                  Amazon Data Firehose
                          |
                          v
                     Destination
```

### Transformation Examples

A Lambda function might:

- Remove unnecessary fields
- Standardize field names
- Normalize timestamps
- Convert application-specific formats
- Filter or enrich records

### Example Input

```json
{
  "user": "customer1001",
  "action": "purchase",
  "amount": 250
}
```

### Example Transformed Data

```json
{
  "customerId": "customer1001",
  "eventType": "purchase",
  "transactionAmount": 250
}
```

### Important Concept

Lambda transformation is optional.

If incoming data is already in the required format, Data Firehose can deliver it without custom transformation logic.

---

# 11. Delivering Data to Amazon S3

Amazon S3 is one of the most common Data Firehose destinations.

It provides durable object storage suitable for long-term retention and analytics.

### Architecture

```text
                  Application Logs
                         |
                         v
                  Amazon Data Firehose
                         |
                         v
                      Buffering
                         |
                         v
                     Amazon S3
                         |
                         v
                   Stored Objects
```

### Example

An organization generates application logs continuously.

The logs are sent to Data Firehose.

Data Firehose delivers the records into an S3 bucket.

The organization can then use other AWS analytics services to query or process the stored data.

### Benefits

- Durable storage
- Scalable storage capacity
- Support for long-term retention
- Integration with AWS analytics services
- S3 lifecycle management

### Important Concept

**Data Firehose + Amazon S3 is a common architecture for streaming data ingestion into a data lake.**

---

# 12. Delivering Data to Amazon OpenSearch Service

Amazon OpenSearch Service is another supported destination.

OpenSearch is commonly used for:

- Log analytics
- Search applications
- Operational dashboards
- Event investigation

### Architecture

```text
                   Application Logs
                          |
                          v
                  Amazon Data Firehose
                          |
                          v
                 Amazon OpenSearch
                          |
                          v
                  Search and Analysis
```

### Example

A distributed application generates operational logs.

The organization wants to search those logs and investigate application problems.

Data Firehose can deliver supported records to Amazon OpenSearch Service.

### Benefits

- Searchable log data
- Operational analytics
- Integration with dashboards
- Reduced custom ingestion development

---

# 13. Delivering Data to Amazon Redshift

Amazon Redshift is a managed cloud data warehousing service.

Organizations use Redshift to analyze large datasets.

### Important Architecture Requirement

The lecture highlights that Data Firehose delivers records to Redshift through an intermediate S3 bucket.

### Architecture

```text
                   Streaming Data
                         |
                         v
                  Amazon Data Firehose
                         |
                         v
                  Intermediate S3 Bucket
                         |
                         v
                   Redshift COPY
                         |
                         v
                   Amazon Redshift
                         |
                         v
                   Data Analytics
```

### How It Works

1. Data Firehose receives streaming records.
2. Records are buffered.
3. Data Firehose writes the records to an S3 bucket.
4. Data Firehose initiates loading into Redshift using the COPY operation.
5. Redshift stores the data for analytical queries.

### Why Use S3?

Amazon S3 acts as the staging location for the data before it is loaded into Redshift.

### AWS Exam Point

**Amazon Data Firehose → Amazon S3 → Amazon Redshift**

Remember the intermediate S3 staging step.

---

# 14. Third-Party Integrations

Amazon Data Firehose can deliver streaming data to supported third-party services.

The lecture specifically mentions Splunk.

It also discusses HTTP and HTTPS endpoints.

### Architecture

```text
                   Application Events
                           |
                           v
                    Amazon Data Firehose
                           |
                +----------+----------+
                |                     |
                v                     v
              Splunk             HTTP Endpoint
```

### Example: Splunk Integration

A company wants to analyze application logs using Splunk.

Instead of maintaining a custom delivery application, the company can configure Data Firehose to send supported log records to Splunk.

### Common Use Cases

- Log analytics
- Security monitoring
- Operational monitoring
- Centralized event collection
- Integration with external analytics platforms

### Important Concept

Data Firehose supports both AWS destinations and supported third-party endpoints.

---

# 15. Amazon Data Firehose vs Kinesis Data Streams

This is the most important comparison in the lesson.

| Feature | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| Primary Purpose | Real-time streaming | Managed data delivery |
| Data Producers | Supported | Supported |
| Data Consumers | Custom and managed consumers | Managed destination delivery |
| Shards | Core capacity concept | No user-managed shard model for direct ingestion |
| Data Retention | 24 hours to 365 days | Temporary buffering and delivery retries |
| Record Replay | Supported within retention | Not a general-purpose replay service |
| Data Transformation | Through consumer applications or integrations | Optional Lambda transformation |
| Amazon S3 Delivery | Requires a consumer or integration | Built-in destination |
| Amazon Redshift Delivery | Requires an integration | Supported through S3 staging |
| OpenSearch Delivery | Requires an integration | Built-in destination |
| Third-Party Delivery | Requires an integration | Supported destinations |
| Latency | Low-latency streaming | Commonly near-real-time |
| Main Benefit | Real-time processing and replay | Simplified data delivery |

### Key Difference

Kinesis Data Streams provides a durable streaming platform with configurable retention.

Amazon Data Firehose provides managed delivery to supported destinations.

---

# 16. When Should You Use Each Service?

### Scenario A – Real-Time Fraud Detection

A financial application must analyze transactions immediately after they occur.

**Recommended Service:** Kinesis Data Streams

### Scenario B – Store Application Logs in S3

A company wants to continuously deliver application logs to Amazon S3.

**Recommended Service:** Amazon Data Firehose

### Scenario C – Real-Time Analytics and Long-Term Storage

A company needs real-time processing and long-term storage.

**Recommended Architecture:**

```text
                    Applications
                         |
                         v
                 Kinesis Data Streams
                         |
              +----------+----------+
              |                     |
              v                     v
       Real-Time Consumer      Amazon Data Firehose
              |                     |
              v                     v
          Analytics              Amazon S3
```

### Scenario D – Load Streaming Data into Redshift

A company wants to continuously load streaming records into a data warehouse using the managed Firehose delivery path.

**Recommended Architecture:**

```text
Streaming Data
      |
      v
Amazon Data Firehose
      |
      v
Amazon S3
      |
      v
Amazon Redshift
```

---

# 17. Real-World Example: Centralized Application Logging

Consider a company running multiple applications.

Each application generates log data.

The organization wants to collect those logs and store them in Amazon S3.

### Architecture

```text
                     APPLICATION SERVERS
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
              App 1        App 2        App 3
                 |            |            |
                 +------------+------------+
                              |
                              v
                     Log Collection
                              |
                              v
                     Amazon Data Firehose
                              |
                              v
                         Buffering
                              |
                              v
                         Amazon S3
                              |
                              v
                       Historical Logs
```

### Benefits

- Centralized log collection
- Managed delivery
- Durable storage
- Reduced operational overhead
- Support for future analytics

---

# 18. Real-World Example: E-Commerce Analytics

An e-commerce platform generates customer interaction events.

The company wants to analyze these events and retain historical records.

### Architecture

```text
                     E-Commerce Website
                              |
                              v
                       Customer Events
                              |
                              v
                     Kinesis Data Streams
                              |
                 +------------+------------+
                 |                         |
                 v                         v
          Real-Time Consumer        Amazon Data Firehose
                 |                         |
                 v                         v
          Live Analytics               Amazon S3
                                           |
                                           v
                                    Historical Analytics
```

### Benefits

- Real-time customer analytics
- Historical data retention
- Separation of processing and storage
- Scalable streaming architecture

---

# 19. Real-World Example: Log Analytics with Splunk

A company wants to collect application logs and analyze them using Splunk.

### Architecture

```text
                   Application Servers
                            |
                            v
                       Log Events
                            |
                            v
                    Amazon Data Firehose
                            |
                            v
                          Splunk
                            |
                            v
                     Log Investigation
```

### Benefits

- Managed log delivery
- Integration with existing monitoring platforms
- Reduced custom integration development
- Centralized operational analytics

---

# 20. Common Mistakes

### Mistake 1: Confusing Data Firehose with Kinesis Data Streams

Kinesis Data Streams is primarily a streaming platform.

Data Firehose is primarily a managed delivery service.

### Mistake 2: Assuming Data Firehose Requires Kinesis Data Streams

Applications can send records directly to Data Firehose.

### Mistake 3: Assuming Data Firehose Provides the Same Replay Capabilities as Kinesis Data Streams

Data Firehose is not intended to provide general-purpose record replay.

### Mistake 4: Assuming Every Destination Receives Records Immediately

Delivery behavior depends on buffering and destination configuration.

### Mistake 5: Forgetting the Intermediate S3 Bucket for Redshift

The Firehose-to-Redshift delivery path uses S3 staging.

### Mistake 6: Assuming Lambda Transformation Is Mandatory

Transformation is optional.

### Mistake 7: Assuming Data Firehose Is Permanent Storage

Data Firehose delivers records to storage or analytics destinations.

The destination provides the persistent storage capability.

### Mistake 8: Assuming Buffering Is Always Exactly 1 MiB or 60 Seconds

Buffering settings depend on the destination and configuration.

---

# 21. AWS Certification Exam Scenarios

## Scenario 1: Deliver Streaming Data to S3

A company generates application logs continuously.

The company wants to deliver those logs to Amazon S3 without maintaining custom consumer applications.

**Question:** Which AWS service should be used?

**Answer:** Amazon Data Firehose.

**Reason:** Data Firehose provides managed streaming data delivery to Amazon S3.

---

## Scenario 2: Real-Time Data Processing

A financial institution needs to analyze transaction events with low latency.

**Question:** Which AWS service is most suitable?

**Answer:** Amazon Kinesis Data Streams.

**Reason:** Kinesis Data Streams supports low-latency streaming applications.

---

## Scenario 3: Deliver Data to Redshift

A company wants to load streaming records into Amazon Redshift using Data Firehose.

**Question:** What intermediate service is required?

**Answer:** Amazon S3.

**Reason:** Data Firehose uses S3 staging and the Redshift COPY operation.

---

## Scenario 4: Transform Streaming Records

A company needs to modify incoming JSON records before delivering them to S3.

**Question:** Which AWS service can perform the transformation?

**Answer:** AWS Lambda integrated with Amazon Data Firehose.

---

## Scenario 5: Deliver Data to Splunk

A company wants to send application logs to Splunk through a managed AWS delivery service.

**Question:** Which service should be used?

**Answer:** Amazon Data Firehose.

---

## Scenario 6: Real-Time Processing and Archival

A company needs to process streaming records in real time and store them in Amazon S3 for historical analysis.

**Question:** Which architecture is suitable?

**Answer:** Kinesis Data Streams with a real-time consumer and Amazon Data Firehose delivering to S3.

---

## Scenario 7: Avoid Managing Consumer Applications

A company wants to continuously deliver streaming data to Amazon OpenSearch Service.

The operations team wants to minimize custom ingestion code.

**Question:** Which service should be used?

**Answer:** Amazon Data Firehose.

---

## Scenario 8: Replay Streaming Records

A company wants to replay previously ingested records within a configurable retention period.

**Question:** Which service is most suitable?

**Answer:** Amazon Kinesis Data Streams.

**Reason:** Kinesis Data Streams retains records for replay within the configured retention period.

---

# 22. Interview Questions

### Q1. What is Amazon Data Firehose?

Amazon Data Firehose is a fully managed service for delivering streaming data to supported destinations.

### Q2. What was Amazon Data Firehose previously called?

Amazon Kinesis Data Firehose.

### Q3. What is the primary difference between Kinesis Data Streams and Data Firehose?

Kinesis Data Streams focuses on real-time streaming and record replay, while Data Firehose focuses on managed delivery to destinations.

### Q4. Does Data Firehose require Kinesis Data Streams?

No. Applications can send data directly to Data Firehose.

### Q5. Can Kinesis Data Streams act as a source for Data Firehose?

Yes.

### Q6. What are common Data Firehose destinations?

Amazon S3, Amazon Redshift, Amazon OpenSearch Service, Splunk, and supported HTTP endpoints.

### Q7. What is buffering in Data Firehose?

Buffering temporarily accumulates records before delivering them to a destination.

### Q8. Why does Data Firehose commonly provide near-real-time delivery?

Because delivery often involves buffering.

### Q9. Can Data Firehose transform incoming records?

Yes, using supported transformation features such as AWS Lambda.

### Q10. Is Lambda transformation mandatory?

No. It is optional.

### Q11. Can Data Firehose deliver data directly to Amazon S3?

Yes.

### Q12. How does Data Firehose deliver records to Amazon Redshift?

It stages data in Amazon S3 and uses the Redshift COPY operation.

### Q13. Can Data Firehose integrate with Splunk?

Yes.

### Q14. Does Data Firehose provide long-term record replay?

No. It is primarily a delivery service rather than a general-purpose replay platform.

### Q15. Which service is better for real-time fraud detection?

Kinesis Data Streams.

### Q16. Which service is better for managed log delivery to S3?

Amazon Data Firehose.

### Q17. Can Data Firehose deliver data to OpenSearch?

Yes, Amazon OpenSearch Service is a supported destination.

### Q18. What is the benefit of combining Kinesis Data Streams with Data Firehose?

The architecture supports real-time processing and managed delivery to persistent storage.

### Q19. Does Data Firehose permanently store incoming data?

No. It delivers data to configured destinations, which may provide persistent storage.

### Q20. What is the main operational advantage of Data Firehose?

It reduces the need to build and maintain custom data delivery applications.

---

# 23. Key Takeaways

- Amazon Data Firehose was previously called Amazon Kinesis Data Firehose.
- It is a fully managed streaming data delivery service.
- It delivers streaming records to supported destinations.
- Amazon S3 is a common destination.
- Amazon Redshift is supported through intermediate S3 staging.
- Amazon OpenSearch Service is a supported destination.
- Splunk and supported HTTP endpoints can receive streaming data.
- Data Firehose can ingest records directly from applications.
- Kinesis Data Streams can also act as a source.
- Data Firehose commonly buffers incoming records.
- Buffering can introduce delivery latency.
- Data Firehose commonly provides near-real-time delivery.
- Buffering settings depend on the destination.
- AWS Lambda can optionally transform incoming records.
- Data Firehose reduces the need for custom consumer applications.
- Kinesis Data Streams supports real-time processing and record replay.
- Data Firehose focuses on managed delivery.
- Combining both services supports real-time analytics and long-term storage.

---

# 24. The Most Important Concepts to Remember

```text
                 AMAZON DATA FIREHOSE
                 ====================

                       DATA SOURCES
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
         Applications   CloudWatch     Kinesis
                           Logs      Data Streams
              |             |             |
              +-------------+-------------+
                            |
                            v
                    AMAZON DATA FIREHOSE
                            |
                            v
                         BUFFERING
                            |
                            v
                  OPTIONAL TRANSFORMATION
                            |
                            v
                       DESTINATION
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
           Amazon S3    OpenSearch      Splunk
              |
              v
         Long-Term Storage


                REDSHIFT DELIVERY PATH
                ======================

                    Data Firehose
                         |
                         v
                      Amazon S3
                         |
                         v
                    Redshift COPY
                         |
                         v
                    Amazon Redshift
```

### AWS Exam Memory Points

| Concept | Remember |
|---|---|
| Amazon Data Firehose | Managed streaming data delivery |
| Previous Name | Kinesis Data Firehose |
| Kinesis Data Streams | Real-time streaming |
| Data Firehose | Commonly near-real-time delivery |
| Buffering | Accumulates records before delivery |
| Buffering Threshold | Size or time, depending on destination |
| Lambda | Optional data transformation |
| Amazon S3 | Persistent object storage destination |
| Amazon Redshift | Delivery through S3 staging |
| Amazon OpenSearch | Search and analytics destination |
| Splunk | Supported third-party destination |
| Direct Ingestion | Supported |
| Kinesis as Source | Supported |
| Replay | Use Kinesis Data Streams |
| Long-Term Storage | Use an appropriate destination |
| Operational Benefit | Less custom delivery infrastructure |

---

# 25. Related Topics

- Amazon Kinesis Data Streams
- Amazon Data Firehose
- Real-Time Data Streaming
- Near-Real-Time Data Delivery
- Streaming Data Architecture
- Data Lakes
- Amazon S3
- Amazon Redshift
- Amazon OpenSearch Service
- AWS Lambda
- Amazon CloudWatch Logs
- Splunk
- Data Transformation
- Data Buffering
- AWS Data Analytics
