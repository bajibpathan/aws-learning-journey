# AWS Glue – Fundamentals and Architecture

> **AWS Glue** is a serverless data integration service that helps organizations discover, prepare, transform, and integrate data from multiple sources for analytics, machine learning, and application development.

---

# 1. Overview

Organizations collect data from multiple systems, including databases, applications, storage services, and streaming platforms.

However, the data is often stored in different formats and structures.

Before organizations can analyze this information, they may need to:

1. Discover where the data is stored.
2. Understand its structure.
3. Extract it from the source.
4. Clean and transform it.
5. Load the transformed data into a destination.

This process is commonly known as **ETL — Extract, Transform, Load**.

AWS Glue provides managed capabilities for building these data integration workflows.

### Topics Covered

1. What is AWS Glue?
2. Understanding ETL
3. AWS Glue architecture
4. Data sources and destinations
5. AWS Glue Crawlers
6. AWS Glue Data Catalog
7. AWS Glue Databases and Tables
8. AWS Glue ETL Jobs
9. Apache Spark integration
10. AWS Glue Streaming ETL
11. Job triggers and scheduling
12. AWS Glue and Amazon EventBridge
13. Security and IAM
14. Real-world use cases
15. AWS Glue vs other analytics services
16. AWS certification scenarios
17. Interview questions

---

# 2. What Is AWS Glue?

AWS Glue is a serverless data integration service.

It enables organizations to discover, prepare, transform, and move data between supported sources and destinations.

### Common Use Cases

- Data preparation for analytics
- Data lake integration
- Database migration and transformation workflows
- Data cleaning
- Data format conversion
- Streaming data transformation
- Machine learning data preparation

### Basic Architecture

```text
                     DATA SOURCES
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Amazon S3    Amazon RDS    DynamoDB
             |            |            |
             +------------+------------+
                          |
                          v
                       AWS GLUE
                          |
                          v
                 EXTRACT / TRANSFORM
                          |
                          v
                     DATA TARGETS
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Amazon S3    Amazon RDS    Redshift
```

### Important Concept

AWS Glue helps automate data integration without requiring organizations to manage the underlying ETL infrastructure.

---

# 3. Understanding ETL

ETL stands for:

**Extract → Transform → Load**

It describes the process of moving data from source systems into destinations after applying the required transformations.

## Step 1 – Extract

Data is retrieved from one or more sources.

### Examples

- Extract customer records from Amazon RDS.
- Read application logs from Amazon S3.
- Retrieve data from DynamoDB.
- Read events from a streaming platform.

## Step 2 – Transform

The extracted data is processed according to business requirements.

### Examples

- Remove duplicate records.
- Standardize date formats.
- Convert data types.
- Filter unnecessary records.
- Combine datasets.
- Clean invalid values.

## Step 3 – Load

The transformed data is written to the destination.

### Examples

- Amazon S3
- Amazon Redshift
- Amazon RDS
- Supported JDBC databases

### Architecture

```text
                  SOURCE DATA
                       |
                       v
                  +---------+
                  | EXTRACT |
                  +---------+
                       |
                       v
                  +-----------+
                  | TRANSFORM |
                  +-----------+
                       |
                       v
                  +------+
                  | LOAD |
                  +------+
                       |
                       v
                  DESTINATION
```

### Example

An organization stores customer records in an operational database.

The records contain inconsistent date formats and duplicate entries.

AWS Glue can extract the records, clean and standardize them, and write the transformed dataset to Amazon S3 for analysis.

---

# 4. AWS Glue Architecture

AWS Glue provides several components that work together.

### Main Components

| Component | Purpose |
|---|---|
| Data Sources | Store the original data |
| Crawlers | Discover schemas and metadata |
| Data Catalog | Stores metadata about datasets |
| ETL Jobs | Extract, transform, and load data |
| Triggers | Initiate jobs or workflows |
| Destinations | Receive transformed data |

### Complete Architecture

```text
                       DATA SOURCES
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          Amazon S3      Amazon RDS      DynamoDB
             |              |              |
             +--------------+--------------+
                            |
                            v
                       GLUE CRAWLER
                            |
                            v
                    DISCOVER METADATA
                            |
                            v
                     GLUE DATA CATALOG
                            |
                            v
                       GLUE ETL JOB
                            |
                            v
                    DATA TRANSFORMATION
                            |
                            v
                       DESTINATIONS
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          Amazon S3     Amazon Redshift   JDBC Database
```

### Processing Flow

1. Source systems store data.
2. A crawler examines supported data sources.
3. The crawler identifies schemas and metadata.
4. The Data Catalog stores the discovered metadata.
5. A Glue ETL job uses the required source definitions and transformation logic.
6. The job reads the actual data from the source.
7. The job transforms the data.
8. The transformed data is written to the destination.

**Important:** A crawler is useful for discovering schemas, but not every Glue job requires a crawler or Data Catalog table.

---

# 5. AWS Glue Data Sources

AWS Glue supports a wide range of data sources.

The lecture highlights both traditional data stores and streaming platforms.

### Common Data Sources

| Source | Description |
|---|---|
| Amazon S3 | Object storage |
| Amazon RDS | Managed relational databases |
| Amazon DynamoDB | NoSQL database |
| JDBC Databases | Supported relational databases |
| Amazon Kinesis Data Streams | Streaming records |
| Apache Kafka | Distributed streaming platform |
| Amazon MSK | Managed Apache Kafka |

### Architecture

```text
                     DATA SOURCES
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
     STORAGE           DATABASES         STREAMS
        |                 |                 |
        v                 v                 v
       S3                RDS              Kinesis
                         DynamoDB        Kafka
                         JDBC            Amazon MSK
        |                 |                 |
        +-----------------+-----------------+
                          |
                          v
                       AWS GLUE
```

### Important Concept

AWS Glue can integrate data from multiple storage, database, and streaming systems.

---

# 6. What Is an AWS Glue Crawler?

An **AWS Glue Crawler** is a managed component that connects to supported data sources, examines their structure, and discovers metadata.

The crawler can create or update table definitions in the AWS Glue Data Catalog.

### Architecture

```text
                      Amazon S3
                          |
                          v
                     GLUE CRAWLER
                          |
                          v
                    EXAMINE DATA
                          |
                          v
                    IDENTIFY SCHEMA
                          |
                          v
                   GLUE DATA CATALOG
```

### What Does a Crawler Discover?

Depending on the source, crawlers can identify information such as:

- Column names
- Data types
- Table structures
- Data formats
- Partition information
- Source locations

### Example

Suppose an S3 bucket contains a CSV file:

```csv
customer_id,customer_name,city,amount
1001,John,Toronto,250
1002,David,Ottawa,175
1003,Sarah,Montreal,300
```

A crawler examines the dataset and infers a schema.

### Example Discovered Schema

| Column | Inferred Data Type |
|---|---|
| customer_id | Integer |
| customer_name | String |
| city | String |
| amount | Integer |

The inferred schema is stored in the Data Catalog.

### Important Clarification

Schema inference is not always perfect.

For example, a crawler may infer an identifier as a number even when the business requires it to be treated as a string.

Production workflows should validate discovered schemas before relying on them.

### AWS Exam Point

**AWS Glue Crawler = Automatically discovers schemas and populates the Glue Data Catalog.**

---

# 7. What Is the AWS Glue Data Catalog?

The **AWS Glue Data Catalog** is a centralized metadata repository.

It stores information describing datasets and their structures.

### Examples of Metadata

- Database definitions
- Table definitions
- Column names
- Column data types
- Data locations
- Partition information
- Other dataset metadata

### Architecture

```text
                      DATA SOURCES
                           |
                           v
                       CRAWLERS
                           |
                           v
                    GLUE DATA CATALOG
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Database     Table        Schema
           Metadata    Metadata     Metadata
```

### Important Concept

**The Glue Data Catalog stores metadata, not the actual source data.**

This distinction is essential for the AWS certification exam.

---

# 8. Metadata vs Actual Data

Consider an Amazon S3 bucket containing customer records.

### Actual Data

```text
s3://company-data/customers/
```

The S3 bucket stores the actual customer dataset.

### Glue Data Catalog

The Data Catalog stores information describing the dataset.

For example:

```text
Database:
customer_analytics

Table:
customers

Location:
s3://company-data/customers/

Columns:
customer_id
customer_name
city
amount
```

### Architecture

```text
                 AMAZON S3
                     |
                     v
              ACTUAL CUSTOMER DATA
                     |
                     |
                     v
                GLUE CRAWLER
                     |
                     v
              GLUE DATA CATALOG
                     |
                     v
                 METADATA
```

### Comparison

| Feature | Amazon S3 | Glue Data Catalog |
|---|---|---|
| Stores Actual Dataset | Yes | No |
| Stores Table Metadata | Not as its primary purpose | Yes |
| Stores Schema Definitions | Not as a centralized catalog | Yes |
| Used for Data Discovery | Through integrations | Yes |
| Stores Data Location | Contains the objects | Records location metadata |

### AWS Exam Point

If a question asks which AWS service provides a **central metadata repository for datasets**, the answer is the **AWS Glue Data Catalog**.

---

# 9. Glue Databases and Tables

AWS Glue uses databases and tables to organize metadata.

These are logical objects within the Data Catalog.

## Glue Database

A Glue database is a logical container for table definitions.

### Example

```text
Glue Data Catalog
        |
        v
Database: customer_analytics
        |
        +------------------+
        |                  |
        v                  v
Table: customers      Table: orders
```

## Glue Table

A Glue table describes a dataset.

It may contain:

- Dataset location
- Column definitions
- Data types
- Storage format
- Partition information

### Example

```text
Database:
customer_analytics

Table:
orders

Location:
s3://company-data/orders/

Schema:
order_id
customer_id
order_date
order_amount
```

### Important Concept

A Glue database is not necessarily a physical relational database.

A Glue table does not necessarily contain the actual records.

These objects describe datasets through metadata.

---

# 10. What Is an AWS Glue ETL Job?

An **AWS Glue ETL Job** is a managed data processing workload that reads data from sources, applies transformations, and writes the results to destinations.

### Architecture

```text
                     SOURCE DATA
                          |
                          v
                      GLUE JOB
                          |
             +------------+------------+
             |            |            |
             v            v            v
           Extract      Transform      Load
                          |
                          v
                     DESTINATION
```

### Common Transformations

- Data cleaning
- Filtering
- Deduplication
- Data type conversion
- Data normalization
- Dataset joins
- Data format conversion

### Example

An organization stores sales data in Amazon S3.

The company wants to remove invalid transactions and standardize the dataset.

A Glue ETL job can:

1. Read sales records from S3.
2. Remove invalid records.
3. Convert fields into the required data types.
4. Write the transformed data to another S3 location.

### Important Concept

**Glue ETL jobs process the actual data.**

The Data Catalog provides metadata that jobs can use to understand source and destination datasets.

---

# 11. AWS Glue and Apache Spark

The lecture explains that AWS Glue can run ETL scripts using Apache Spark.

Apache Spark is a distributed data processing engine.

It is commonly used to process large datasets.

### Architecture

```text
                    DATA SOURCES
                         |
                         v
                    AWS GLUE JOB
                         |
                         v
                    APACHE SPARK
                         |
                         v
                 DATA TRANSFORMATION
                         |
                         v
                     DESTINATION
```

### Why Use Apache Spark?

Apache Spark supports:

- Distributed data processing
- Large-scale transformations
- Parallel processing
- Data filtering
- Data aggregation
- Dataset joins

### Example

A company stores millions of transaction records.

A Spark-based Glue job can distribute the processing workload across managed compute resources.

### Important Concept

AWS Glue provides managed execution environments for supported ETL workloads, including Spark-based jobs.

---

# 12. AWS Glue ETL Job Execution

Glue jobs can be started through different mechanisms.

### Execution Options

| Method | Description |
|---|---|
| Manual | Start a job on demand |
| Scheduled | Run a job according to a schedule |
| Event-Driven | Start processing in response to an event |
| Workflow-Based | Coordinate jobs and crawlers |

### Architecture

```text
                    JOB TRIGGERS
                         |
           +-------------+-------------+
           |             |             |
           v             v             v
         Manual       Scheduled     Event-Driven
           |             |             |
           +-------------+-------------+
                         |
                         v
                     GLUE JOB
                         |
                         v
                   TRANSFORM DATA
                         |
                         v
                     DESTINATION
```

### Important Concept

Glue jobs do not need to run continuously for batch ETL workloads.

They can execute only when required.

---

# 13. AWS Glue Streaming ETL

AWS Glue also supports streaming ETL workloads.

Unlike traditional batch processing, streaming ETL continuously processes incoming records.

### Supported Streaming Sources Mentioned in the Lecture

- Amazon Kinesis Data Streams
- Apache Kafka
- Amazon Managed Streaming for Apache Kafka (Amazon MSK)

### Architecture

```text
                    STREAMING SOURCES
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Kinesis       Kafka       Amazon MSK
              |            |            |
              +------------+------------+
                           |
                           v
                    GLUE STREAMING ETL
                           |
                           v
                  APACHE SPARK STRUCTURED
                        STREAMING
                           |
                           v
                    CLEAN / TRANSFORM
                           |
                           v
                      DESTINATION
```

### How Streaming ETL Works

1. Streaming sources continuously generate records.
2. A Glue streaming job reads the incoming records.
3. Apache Spark Structured Streaming processes the data.
4. Transformations are applied.
5. Processed records are written to supported destinations.

### Example

An IoT platform receives temperature readings from thousands of devices.

The organization wants to clean the readings and store the processed results in S3.

A Glue streaming ETL job can continuously read the records, transform them, and write the results.

### AWS Exam Point

**AWS Glue Streaming ETL uses Apache Spark Structured Streaming and can process data from Kinesis Data Streams, Kafka, and Amazon MSK.**

---

# 14. Batch ETL vs Streaming ETL

| Feature | Batch ETL | Streaming ETL |
|---|---|---|
| Processing Model | Processes datasets in batches | Processes continuously arriving data |
| Typical Execution | Scheduled or on demand | Long-running |
| Data Sources | Stored datasets | Streaming platforms |
| Common Engine | Apache Spark | Spark Structured Streaming |
| Example | Daily sales processing | Continuous IoT processing |
| Typical Requirement | Periodic data preparation | Continuous transformation |

### Batch Architecture

```text
Amazon S3
    |
    v
Scheduled Glue Job
    |
    v
Transform Dataset
    |
    v
Amazon S3
```

### Streaming Architecture

```text
Kinesis Data Streams
        |
        v
Glue Streaming ETL
        |
        v
Continuous Transformation
        |
        v
Amazon S3
```

### Important Concept

Batch ETL processes data at defined intervals.

Streaming ETL processes continuously arriving records.

---

# 15. AWS Glue and Amazon EventBridge

The lecture explains that Glue jobs can be initiated through events and schedules.

Amazon EventBridge can help integrate Glue workflows with event-driven architectures.

### Example Architecture

```text
                   APPLICATION EVENT
                          |
                          v
                    AMAZON EVENTBRIDGE
                          |
                          v
                      MATCHING RULE
                          |
                          v
                       GLUE JOB
                          |
                          v
                    TRANSFORM DATA
                          |
                          v
                       AMAZON S3
```

### Example

A company uploads a new dataset to an S3 bucket.

An event-driven workflow can initiate a Glue ETL job to process the uploaded data.

### Scheduled Processing

A company may also need to process data every night.

```text
                    DAILY SCHEDULE
                          |
                          v
                     EVENTBRIDGE
                          |
                          v
                       GLUE JOB
                          |
                          v
                    PROCESS DATA
```

### Important Clarification

EventBridge integration depends on the selected target and workflow configuration.

For some workflows, an intermediate service such as AWS Lambda or AWS Step Functions may be used to start or coordinate Glue jobs.

### AWS Exam Point

Use scheduled or event-driven workflows when ETL processing should occur automatically.

---

# 16. AWS Glue Security and IAM

The AWS Glue Data Catalog may be accessed by multiple teams and services.

However, access should be controlled.

### Common Security Requirements

- Restrict access to Glue databases.
- Restrict access to table metadata.
- Control who can create or modify crawlers.
- Control who can start Glue jobs.
- Grant jobs access only to required data sources.
- Grant jobs permission to write only to authorized destinations.

### Architecture

```text
                    AWS IAM
                       |
                       v
                ACCESS CONTROL
                       |
            +----------+----------+
            |          |          |
            v          v          v
         Crawlers   Data Catalog  ETL Jobs
```

### Example

A data analyst may require permission to view metadata but not modify Glue jobs.

An ETL execution role may require access to:

- Read from an S3 source bucket
- Retrieve required Data Catalog metadata
- Write to an S3 destination bucket
- Write operational logs

### Important Concept

IAM permissions should follow the principle of least privilege.

**Additional AWS context:** AWS Lake Formation can also provide fine-grained data access controls in supported data lake architectures.

---

# 17. AWS Glue vs Amazon Kinesis Data Streams

These services solve different problems.

| Feature | AWS Glue | Kinesis Data Streams |
|---|---|---|
| Primary Purpose | Data integration and ETL | Real-time data streaming |
| Data Discovery | Crawlers and metadata catalog | Not its primary purpose |
| Data Transformation | Core capability | Performed by consumers |
| Streaming Support | Streaming ETL | Native streaming platform |
| Record Retention | Depends on source and destination | Configurable stream retention |
| Main Use Case | Prepare and integrate data | Ingest and distribute streaming records |

### Example

Kinesis Data Streams receives continuous IoT readings.

AWS Glue Streaming ETL transforms those readings.

Amazon S3 stores the processed data.

```text
IoT Devices
     |
     v
Kinesis Data Streams
     |
     v
AWS Glue Streaming ETL
     |
     v
Amazon S3
```

---

# 18. AWS Glue vs Amazon Data Firehose

AWS Glue and Amazon Data Firehose can both participate in data processing pipelines.

However, their primary responsibilities differ.

| Feature | AWS Glue | Amazon Data Firehose |
|---|---|---|
| Primary Purpose | Data integration and transformation | Managed streaming delivery |
| Schema Discovery | Supported through crawlers | Not its primary purpose |
| Data Catalog | Yes | No dedicated equivalent |
| ETL Processing | Extensive transformations | Optional transformation capabilities |
| Batch Processing | Supported | Not its primary purpose |
| Streaming Processing | Supported | Streaming delivery |
| Amazon S3 Destination | Supported | Supported |
| Main Benefit | Flexible data preparation | Simplified data delivery |

### Example

```text
APPLICATION LOGS
       |
       v
AMAZON DATA FIREHOSE
       |
       v
AMAZON S3
       |
       v
AWS GLUE CRAWLER
       |
       v
GLUE DATA CATALOG
       |
       v
AWS GLUE ETL JOB
       |
       v
TRANSFORMED DATA
       |
       v
AMAZON S3
```

### Important Concept

Data Firehose delivers incoming records.

Glue can discover the resulting datasets and perform additional transformations.

---

# 19. Real-World Example: Data Lake Processing

A company collects sales data from multiple applications.

The data is stored in Amazon S3.

The organization wants to clean the data and prepare it for analytics.

### Architecture

```text
                    SALES APPLICATIONS
                            |
                            v
                         AMAZON S3
                            |
                            v
                       GLUE CRAWLER
                            |
                            v
                     GLUE DATA CATALOG
                            |
                            v
                       GLUE ETL JOB
                            |
                            v
                     CLEAN SALES DATA
                            |
                            v
                         AMAZON S3
                            |
                            v
                     ANALYTICS SERVICES
```

### Benefits

- Automated schema discovery
- Centralized metadata
- Data cleaning
- Reusable ETL workflows
- Prepared datasets for analytics

---

# 20. Real-World Example: Database Data Integration

A company stores customer data in Amazon RDS.

The organization wants to extract the records, transform them, and load them into a data warehouse.

### Architecture

```text
                      AMAZON RDS
                          |
                          v
                     GLUE CRAWLER
                          |
                          v
                    GLUE DATA CATALOG
                          |
                          v
                      GLUE ETL JOB
                          |
                          v
                   TRANSFORM RECORDS
                          |
                          v
                    AMAZON REDSHIFT
```

### Benefits

- Managed ETL processing
- Data standardization
- Integration between operational and analytical systems
- Reduced infrastructure management

---

# 21. Real-World Example: IoT Streaming ETL

An organization receives continuous sensor readings.

The company wants to clean and transform these readings before storing them.

### Architecture

```text
                       IoT DEVICES
                           |
                           v
                    SENSOR READINGS
                           |
                           v
                  KINESIS DATA STREAMS
                           |
                           v
                   GLUE STREAMING ETL
                           |
                           v
                   CLEAN / TRANSFORM
                           |
                           v
                       AMAZON S3
```

### Benefits

- Continuous data processing
- Scalable streaming transformation
- Managed processing infrastructure
- Integration with persistent storage

---

# 22. Common Mistakes

### Mistake 1: Assuming the Glue Data Catalog Stores Actual Data

The Data Catalog stores metadata describing datasets.

### Mistake 2: Confusing Crawlers with ETL Jobs

Crawlers discover schemas.

ETL jobs process and transform data.

### Mistake 3: Assuming Every Glue Job Requires a Crawler

Glue jobs can be configured without first running a crawler.

### Mistake 4: Assuming Glue Only Supports Batch Processing

AWS Glue also supports streaming ETL.

### Mistake 5: Confusing Glue with Kinesis Data Streams

Kinesis Data Streams ingests streaming records.

Glue performs data integration and transformation.

### Mistake 6: Assuming Glue Databases Are Physical Databases

Glue databases are logical metadata containers in the Data Catalog.

### Mistake 7: Forgetting Job Execution Permissions

Glue jobs need appropriate permissions to read sources and write destinations.

### Mistake 8: Assuming Crawlers Always Infer the Correct Schema

Inferred schemas should be validated when accuracy is important.

---

# 23. AWS Certification Exam Scenarios

## Scenario 1: Discover Data Schemas

A company stores thousands of CSV and JSON files in Amazon S3.

The company wants to automatically discover their schemas.

**Question:** Which AWS service should be used?

**Answer:** AWS Glue Crawlers.

**Reason:** Crawlers examine supported data sources and populate metadata in the Glue Data Catalog.

---

## Scenario 2: Centralized Metadata Repository

A company wants a centralized repository containing database and table definitions for its datasets.

**Question:** Which AWS service provides this capability?

**Answer:** AWS Glue Data Catalog.

---

## Scenario 3: Transform Data Before Analytics

A company needs to extract records from Amazon RDS, clean them, and write the transformed dataset to Amazon S3.

**Question:** Which service is suitable?

**Answer:** AWS Glue ETL.

---

## Scenario 4: Streaming ETL

A company needs to continuously process records from Amazon Kinesis Data Streams.

**Question:** Which AWS service supports this requirement?

**Answer:** AWS Glue Streaming ETL.

---

## Scenario 5: Apache Kafka Integration

A company uses Apache Kafka and needs to transform streaming records.

**Question:** Which AWS Glue capability can be used?

**Answer:** AWS Glue Streaming ETL.

---

## Scenario 6: Scheduled Data Processing

A company needs to execute an ETL job every night.

**Question:** How can this be automated?

**Answer:** Configure a scheduled Glue trigger or an appropriate EventBridge scheduling workflow.

---

## Scenario 7: Event-Driven ETL

A company wants to start data processing when a new dataset becomes available.

**Question:** Which architecture is suitable?

**Answer:** An event-driven workflow using EventBridge and AWS Glue.

---

## Scenario 8: Metadata vs Data

A company uses an AWS Glue crawler to examine files stored in S3.

**Question:** Where is the actual data stored?

**Answer:** Amazon S3.

**Question:** Where is the discovered metadata stored?

**Answer:** AWS Glue Data Catalog.

---

## Scenario 9: Prepare Machine Learning Data

A company needs to clean and transform datasets before using them in machine learning workflows.

**Question:** Which AWS service is suitable?

**Answer:** AWS Glue.

---

## Scenario 10: Minimize ETL Infrastructure Management

A company wants to perform large-scale ETL without maintaining its own processing servers.

**Question:** Which service should it consider?

**Answer:** AWS Glue.

**Reason:** AWS Glue provides serverless, managed data integration capabilities.

---

# 24. Interview Questions

### Q1. What is AWS Glue?

AWS Glue is a serverless data integration service used to discover, prepare, transform, and integrate data.

### Q2. What does ETL stand for?

Extract, Transform, Load.

### Q3. What is an AWS Glue Crawler?

A managed component that examines supported data sources and discovers schemas and metadata.

### Q4. What is the AWS Glue Data Catalog?

A centralized metadata repository containing database and table definitions and other dataset metadata.

### Q5. Does the Glue Data Catalog store actual data?

No. It stores metadata describing datasets.

### Q6. What is a Glue database?

A logical container for metadata tables in the Data Catalog.

### Q7. What is a Glue table?

A metadata definition describing a dataset's schema, location, and other properties.

### Q8. What is a Glue ETL job?

A managed processing workload that extracts, transforms, and loads data.

### Q9. Which processing engine is commonly used by AWS Glue?

Apache Spark.

### Q10. Does AWS Glue support streaming ETL?

Yes.

### Q11. Which engine supports Glue streaming ETL?

Apache Spark Structured Streaming.

### Q12. Which streaming sources are mentioned in the lecture?

Amazon Kinesis Data Streams, Apache Kafka, and Amazon MSK.

### Q13. Can Glue read data from Amazon S3?

Yes.

### Q14. Can Glue integrate with Amazon RDS?

Yes, using supported connections and configurations.

### Q15. Can Glue transform data before loading it into Redshift?

Yes.

### Q16. How can Glue jobs be started?

Manually, through schedules, or through supported event-driven workflows.

### Q17. What is the difference between a crawler and an ETL job?

A crawler discovers metadata, while an ETL job processes actual data.

### Q18. What is the difference between AWS Glue and Kinesis Data Streams?

Glue performs data integration and transformation, while Kinesis Data Streams provides real-time streaming ingestion.

### Q19. What is the difference between AWS Glue and Amazon Data Firehose?

Glue provides broader ETL capabilities, while Data Firehose focuses on managed streaming delivery.

### Q20. Why is IAM important for AWS Glue?

IAM controls access to Glue resources and the data sources and destinations used by Glue jobs.

---

# 25. Key Takeaways

- AWS Glue is a serverless data integration service.
- It supports Extract, Transform, and Load operations.
- Glue integrates with multiple data sources.
- Amazon S3, RDS, DynamoDB, and JDBC databases are common sources.
- Kinesis Data Streams, Kafka, and Amazon MSK can provide streaming data.
- Glue Crawlers discover schemas and metadata.
- The Glue Data Catalog stores metadata.
- The Data Catalog does not store the actual dataset contents.
- Glue databases and tables are logical metadata objects.
- Glue ETL jobs process and transform actual data.
- Apache Spark is commonly used for Glue ETL.
- Glue supports both batch and streaming ETL.
- Streaming ETL uses Apache Spark Structured Streaming.
- Glue jobs can run manually or through automated workflows.
- Amazon EventBridge can participate in event-driven ETL architectures.
- IAM controls access to Glue resources and connected data.
- AWS Glue is commonly used for analytics, machine learning data preparation, and data lake processing.

---

# 26. The Most Important Concepts to Remember

```text
                       AWS GLUE
                       ========

                      DATA SOURCES
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          Amazon S3     Amazon RDS     DynamoDB
             |             |             |
             +-------------+-------------+
                           |
                           v
                      GLUE CRAWLER
                           |
                           v
                     DISCOVER SCHEMA
                           |
                           v
                    GLUE DATA CATALOG
                           |
                           v
                     STORE METADATA
                           |
                           v
                       GLUE JOB
                           |
                           v
                    EXTRACT DATA
                           |
                           v
                   TRANSFORM DATA
                           |
                           v
                      LOAD DATA
                           |
                           v
                       DESTINATION


                 STREAMING ETL ARCHITECTURE
                 ==========================

                     IoT / Clickstreams
                             |
                             v
                    Kinesis Data Streams
                             |
                             v
                     Glue Streaming ETL
                             |
                             v
                    Spark Structured Streaming
                             |
                             v
                        Amazon S3
```

### AWS Exam Memory Points

| Concept | Remember |
|---|---|
| AWS Glue | Serverless data integration |
| ETL | Extract, Transform, Load |
| Glue Crawler | Discovers schemas |
| Glue Data Catalog | Stores metadata |
| Glue Database | Logical metadata container |
| Glue Table | Dataset metadata definition |
| Glue ETL Job | Processes actual data |
| Apache Spark | Distributed processing engine |
| Streaming ETL | Continuous data transformation |
| Spark Structured Streaming | Engine for Glue streaming ETL |
| Kinesis Data Streams | Streaming source |
| Apache Kafka | Streaming source |
| Amazon MSK | Managed Kafka |
| Amazon EventBridge | Event-driven workflow integration |
| IAM | Access control |
| Amazon S3 | Common source and destination |
| Amazon Redshift | Analytical data warehouse |
| Amazon Data Firehose | Managed streaming data delivery |

---

# 27. Related Topics

- AWS Glue
- Extract, Transform, Load
- AWS Glue Crawlers
- AWS Glue Data Catalog
- AWS Glue Databases and Tables
- AWS Glue ETL Jobs
- Apache Spark
- Apache Spark Structured Streaming
- AWS Glue Streaming ETL
- Amazon Kinesis Data Streams
- Amazon Data Firehose
- Amazon S3
- Amazon RDS
- Amazon DynamoDB
- Amazon Redshift
- Amazon EventBridge
- AWS IAM
- Data Lakes
- Data Warehousing
- AWS Data Analytics

