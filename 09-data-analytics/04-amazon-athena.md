# Amazon Athena – Fundamentals and Architecture

> **Amazon Athena** is a serverless, interactive query service that enables users to analyze data stored in Amazon S3 using SQL without provisioning database servers or loading the data into a traditional database.

---

# 1. Overview

Organizations often store large volumes of data in Amazon S3.

This data may include:

- Application logs
- Customer transactions
- Website activity
- Sales records
- Financial reports
- IoT data
- CSV files
- JSON documents
- Parquet files

Businesses frequently need to analyze this information.

Traditionally, organizations might load the data into a relational database or data warehouse before querying it.

However, loading and maintaining another copy of the data introduces additional processing, infrastructure, and operational overhead.

**Amazon Athena provides an alternative.**

It allows users to query data directly in Amazon S3 using SQL.

### Topics Covered

1. What is Amazon Athena?
2. Why use Amazon Athena?
3. Amazon Athena architecture
4. Understanding schemas
5. Schema-on-read
6. External tables
7. AWS Glue Data Catalog integration
8. Supported file formats
9. SQL queries
10. Query performance optimization
11. Data partitioning
12. Columnar formats
13. Compression and file sizes
14. Athena Federated Query
15. Athena vs AWS Glue
16. Athena vs Amazon Redshift
17. Real-world use cases
18. AWS certification scenarios
19. Interview questions

---

# 2. What Is Amazon Athena?

Amazon Athena is an interactive SQL query service.

It allows users to analyze data stored in Amazon S3 without managing database infrastructure.

### Key Characteristics

- Serverless
- Supports SQL queries
- Queries data directly in S3
- Supports multiple file formats
- Uses schema-on-read
- Integrates with AWS Glue Data Catalog
- Supports ad hoc analytics
- Offers usage-based pricing
- Supports federated queries to additional data sources

### Basic Architecture

```text
                    DATA FILES
                        |
           +------------+------------+
           |            |            |
           v            v            v
         CSV           JSON        PARQUET
           |            |            |
           +------------+------------+
                        |
                        v
                    AMAZON S3
                        |
                        v
                   AMAZON ATHENA
                        |
                        v
                     SQL QUERY
                        |
                        v
                   QUERY RESULTS
```

### Important Concept

**Amazon Athena queries data in place rather than requiring the data to be loaded into a traditional database.**

---

# 3. Why Do We Need Amazon Athena?

Consider a company that stores millions of sales transactions in Amazon S3.

The company wants to answer questions such as:

- How many products were sold yesterday?
- Which city generated the highest revenue?
- What was the total monthly revenue?
- Which products are most popular?
- How many customers purchased a specific product?

One approach would be to load the data into a database and then run SQL queries.

However, this may be unnecessary for occasional analysis.

### Traditional Approach

```text
                  AMAZON S3
                      |
                      v
                 EXTRACT DATA
                      |
                      v
                LOAD DATABASE
                      |
                      v
                  RUN SQL
                      |
                      v
                 QUERY RESULTS
```

### Amazon Athena Approach

```text
                  AMAZON S3
                      |
                      v
                  AMAZON ATHENA
                      |
                      v
                    RUN SQL
                      |
                      v
                 QUERY RESULTS
```

### Benefits

- No database server provisioning
- No traditional data-loading requirement
- SQL-based analysis
- Suitable for ad hoc queries
- Integration with existing S3 data lakes
- Pay-per-use options

### AWS Exam Point

When a scenario requires **serverless SQL queries directly against data stored in S3**, consider **Amazon Athena**.

---

# 4. Amazon Athena Architecture

Amazon Athena works with data stored in S3 and metadata describing that data.

### Main Components

| Component | Purpose |
|---|---|
| Amazon S3 | Stores the actual data |
| AWS Glue Data Catalog | Stores dataset metadata |
| External Tables | Describe data structure and location |
| Amazon Athena | Executes SQL queries |
| Query Results Location | Stores query output |

### Architecture

```text
                       AMAZON S3
                           |
              +------------+------------+
              |            |            |
              v            v            v
            CSV          JSON         PARQUET
              |            |            |
              +------------+------------+
                           |
                           v
                      GLUE CRAWLER
                           |
                           v
                    GLUE DATA CATALOG
                           |
                           v
                    TABLE DEFINITIONS
                           |
                           v
                      AMAZON ATHENA
                           |
                           v
                        SQL QUERY
                           |
                           v
                      QUERY RESULTS
```

### Processing Flow

1. Data is stored in Amazon S3.
2. A schema describing the data is created or discovered.
3. Metadata is registered in a catalog.
4. Athena uses the metadata to interpret the dataset.
5. A user executes a SQL query.
6. Athena reads the required data from S3.
7. Athena returns the query results.

**Important:** Athena does not require a Glue crawler for every dataset. Table definitions can also be created manually.

---

# 5. Understanding a Schema

Before understanding schema-on-read, we need to understand what a schema is.

A **schema** defines how data is organized and interpreted.

It can describe:

- Tables
- Columns
- Data types
- Relationships
- Constraints

### Example

Consider the following sales dataset:

| order_id | product_name | city | quantity | amount |
|---|---|---|---:|---:|
| 1001 | Cinnamon Danish | London | 5 | 25 |
| 1002 | Coffee | Toronto | 3 | 15 |
| 1003 | Cinnamon Danish | London | 8 | 40 |

The schema might define:

| Column | Data Type |
|---|---|
| order_id | INTEGER |
| product_name | STRING |
| city | STRING |
| quantity | INTEGER |
| amount | DECIMAL |

### Important Concept

The schema describes the structure of the data.

The actual records contain the values.

---

# 6. What Is Schema-on-Read?

Amazon Athena uses a **schema-on-read** approach.

This means that the data can remain in its original format in S3, while a defined schema is used to interpret the data when queries execute.

### Architecture

```text
                     AMAZON S3
                         |
                         v
                   RAW DATA FILES
                         |
                         v
                    AMAZON ATHENA
                         |
                         v
                    APPLY SCHEMA
                         |
                         v
                     EXECUTE SQL
                         |
                         v
                    QUERY RESULTS
```

### Example

Suppose S3 contains a CSV file:

```csv
order_id,product_name,city,quantity,amount
1001,Cinnamon Danish,London,5,25
1002,Coffee,Toronto,3,15
1003,Cinnamon Danish,London,8,40
```

The original CSV file remains in S3.

Athena uses a table definition to interpret the columns and their data types.

### Key Concept

**Schema-on-read means the schema is applied when the data is read, rather than requiring the data to be loaded into a predefined database structure.**

---

# 7. Schema-on-Read vs Schema-on-Write

This comparison is useful for understanding Athena.

| Feature | Schema-on-Read | Schema-on-Write |
|---|---|---|
| Schema Application | When data is read | Before or during data ingestion |
| Data Storage | Can retain original format | Typically conforms to predefined structure |
| Flexibility | High | More controlled |
| Common Use Case | Data lakes | Traditional relational databases |
| Example | Amazon Athena over S3 | Traditional relational database tables |

### Schema-on-Write

```text
SOURCE DATA
     |
     v
VALIDATE AGAINST SCHEMA
     |
     v
LOAD INTO DATABASE
     |
     v
RUN QUERY
```

### Schema-on-Read

```text
SOURCE DATA
     |
     v
STORE IN AMAZON S3
     |
     v
DEFINE EXTERNAL TABLE
     |
     v
RUN QUERY
```

### AWS Exam Point

**Athena uses schema-on-read for querying S3 datasets.**

---

# 8. What Is an External Table?

An external table describes data stored outside the query engine.

For Athena, external tables commonly describe datasets stored in Amazon S3.

### Example

```text
External Table:
sales

Columns:
order_id
product_name
city
quantity
amount

Location:
s3://company-data/sales/
```

### Architecture

```text
                    AMAZON ATHENA
                         |
                         v
                    EXTERNAL TABLE
                         |
                         v
                  TABLE METADATA
                         |
                         v
                    AMAZON S3
                         |
                         v
                    ACTUAL DATA
```

### Important Concept

The external table contains metadata describing the dataset.

It does not move the original dataset from S3 into Athena.

---

# 9. Creating Tables in Amazon Athena

Athena needs metadata to interpret data stored in S3.

The lecture describes several ways to create this metadata.

### Method 1 – Manual Table Creation

A user can create a table using SQL.

### Method 2 – AWS Glue Crawler

A Glue crawler examines the dataset and populates the Glue Data Catalog.

### Method 3 – Programmatic Creation

Table metadata can also be created through supported APIs and automation.

### Example SQL

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS sales (
    order_id INT,
    product_name STRING,
    city STRING,
    quantity INT,
    amount DECIMAL(10,2)
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://company-data/sales/';
```

**Example assumption:** The S3 location contains compatible CSV data. Header handling and other CSV parsing settings may need additional configuration.

### Important Concept

Athena requires appropriate table metadata, but the actual data remains in S3.

---

# 10. AWS Glue Data Catalog Integration

In the previous lecture, we learned that AWS Glue Crawlers discover schemas and populate the Glue Data Catalog.

Athena can use these metadata definitions.

### Architecture

```text
                      AMAZON S3
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
                     AMAZON ATHENA
                          |
                          v
                       SQL QUERY
```

### Example Workflow

1. Upload CSV files into Amazon S3.
2. Configure an AWS Glue crawler.
3. Run the crawler.
4. The crawler discovers the dataset schema.
5. The Glue Data Catalog stores the table definition.
6. Athena uses the table metadata.
7. Execute SQL queries against the S3 dataset.

### Benefits

- Automated schema discovery
- Centralized metadata
- Reusable table definitions
- Integration between AWS analytics services

### AWS Exam Point

**Glue Crawler → Glue Data Catalog → Athena → SQL Queries**

This is a common data lake analytics architecture.

---

# 11. Supported Data Formats

The lecture identifies several formats that Athena can query.

### Common Formats

| Format | Description |
|---|---|
| CSV | Comma-separated values |
| TSV | Tab-separated values |
| JSON | Semi-structured data |
| Parquet | Columnar storage format |
| ORC | Optimized columnar storage format |

### Architecture

```text
                     AMAZON S3
                         |
            +------------+------------+
            |            |            |
            v            v            v
           CSV          JSON        PARQUET
            |            |            |
            +------------+------------+
                         |
                         v
                     AMAZON ATHENA
                         |
                         v
                      SQL QUERY
```

### Important Concept

Athena supports multiple data formats, allowing organizations to analyze different types of datasets stored in S3.

**Technical clarification:** Athena does not automatically interpret every arbitrary unstructured file. The data must be readable through a supported format, SerDe, connector, or appropriate table definition.

---

# 12. Querying Data Using SQL

Amazon Athena supports SQL queries against registered datasets.

### Example Dataset

| order_id | product_name | city | quantity | amount |
|---|---|---|---:|---:|
| 1001 | Cinnamon Danish | London | 5 | 25 |
| 1002 | Coffee | Toronto | 3 | 15 |
| 1003 | Cinnamon Danish | London | 8 | 40 |
| 1004 | Cinnamon Danish | Ottawa | 4 | 20 |

### Example 1 – Retrieve All Records

```sql
SELECT *
FROM sales;
```

### Example 2 – Filter by City

```sql
SELECT *
FROM sales
WHERE city = 'London';
```

### Example 3 – Calculate Total Sales

```sql
SELECT SUM(amount) AS total_sales
FROM sales;
```

### Example 4 – Calculate Product Quantity

This example follows the coffee shop scenario from the lecture.

```sql
SELECT SUM(quantity) AS total_danishes_sold
FROM sales
WHERE product_name = 'Cinnamon Danish'
  AND city = 'London';
```

### Expected Result

```text
total_danishes_sold
-------------------
13
```

### Example 5 – Group Sales by City

```sql
SELECT
    city,
    SUM(amount) AS total_sales
FROM sales
GROUP BY city;
```

### Example 6 – Sort by Revenue

```sql
SELECT
    city,
    SUM(amount) AS total_sales
FROM sales
GROUP BY city
ORDER BY total_sales DESC;
```

### Important Concept

Athena supports SQL operations such as:

- SELECT
- WHERE
- GROUP BY
- ORDER BY
- JOIN
- Aggregation functions

This makes Athena suitable for both simple and complex analytical queries.

---

# 13. Amazon Athena Pricing Concept

Athena is serverless.

For its commonly used SQL query pricing model, charges are based primarily on the amount of data scanned.

Other pricing models and charges may apply depending on the selected capacity model, features, and associated AWS services.

### Example

Suppose a query scans a large dataset stored in S3.

If the dataset is not optimized, Athena may need to read more data than necessary.

This can increase:

- Query execution time
- Data scanned
- Query cost

### Important Concept

**Reducing unnecessary data scans is a key Athena cost optimization strategy.**

---

# 14. Athena Performance Optimization

The lecture highlights several optimization techniques.

### Recommended Techniques

1. Partition data.
2. Use columnar formats.
3. Compress data.
4. Avoid excessive small files.
5. Optimize file sizes.
6. Read only required columns.

These techniques can improve query performance and reduce scanned data.

---

# 15. Partitioning Data in Amazon S3

Partitioning organizes data into logical groups based on selected column values.

### Example

Suppose an organization stores sales data by year, month, and day.

```text
s3://company-data/sales/
    |
    +-- year=2026/
          |
          +-- month=10/
          |     |
          |     +-- day=01/
          |     +-- day=02/
          |
          +-- month=11/
                |
                +-- day=01/
                +-- day=02/
```

### Why Partition Data?

Suppose a query needs sales data for November 2026.

Without effective partitioning, Athena may scan a much larger dataset.

With partitioning, Athena can limit scanning to the relevant partitions.

### Example Query

```sql
SELECT SUM(amount)
FROM sales
WHERE year = '2026'
  AND month = '11';
```

### Important Concept

**Partition pruning helps Athena avoid scanning unrelated data.**

### AWS Exam Point

If a question asks how to reduce Athena query costs for date-based queries, consider **partitioning S3 data by date**.

---

# 16. Columnar Data Formats

The lecture recommends using columnar formats such as:

- Apache Parquet
- Apache ORC

### Row-Based Storage

```text
Record 1:
order_id, product, city, amount

Record 2:
order_id, product, city, amount

Record 3:
order_id, product, city, amount
```

### Columnar Storage

```text
Column: order_id
1001, 1002, 1003

Column: product
Coffee, Danish, Tea

Column: city
London, Toronto, Ottawa

Column: amount
25, 40, 15
```

### Why Columnar Formats Help

Suppose a query only needs the `amount` column.

A columnar format allows Athena to read the relevant column data rather than unnecessarily reading every column.

### Example

```sql
SELECT SUM(amount)
FROM sales;
```

### Benefits

- Reduced data scanning
- Improved analytical query performance
- Lower query costs
- Efficient compression

### AWS Exam Point

**Parquet and ORC are recommended formats for optimizing Athena analytics workloads.**

---

# 17. Using AWS Glue to Convert Data Formats

AWS Glue can transform raw datasets into more efficient formats.

### Example Architecture

```text
                     AMAZON S3
                         |
                         v
                    RAW CSV FILES
                         |
                         v
                    AWS GLUE ETL
                         |
                         v
                   CONVERT TO PARQUET
                         |
                         v
                     AMAZON S3
                         |
                         v
                    AMAZON ATHENA
                         |
                         v
                       SQL QUERY
```

### Example Workflow

1. Raw CSV files arrive in S3.
2. AWS Glue reads the files.
3. Glue transforms the dataset into Parquet.
4. Glue writes the transformed files to S3.
5. Athena queries the Parquet dataset.

### Benefits

- Lower data scanning
- Better query efficiency
- Reduced storage requirements in many cases
- Improved analytics performance

### Important Concept

**AWS Glue prepares the data; Amazon Athena queries the data.**

---

# 18. Compressing Data

Compression reduces the physical size of stored data.

Smaller files can reduce the amount of data transferred and scanned.

### Example

```text
UNCOMPRESSED DATA
       |
       v
   Large Files
       |
       v
   More Data Read


COMPRESSED DATA
       |
       v
   Smaller Files
       |
       v
   Less Data Read
```

### Benefits

- Reduced storage usage
- Lower data transfer overhead
- Potentially reduced query costs
- Improved processing efficiency

### Important Clarification

The benefit depends on the file format, compression codec, and whether the query engine can efficiently read the compressed data.

---

# 19. Optimizing File Sizes

The lecture recommends avoiding large numbers of very small files.

### Why Are Small Files a Problem?

A dataset containing thousands of tiny files can create unnecessary overhead.

The query engine may need to open and process many individual objects.

### Example

**Less Efficient**

```text
sales/
  file001.csv
  file002.csv
  file003.csv
  ...
  file10000.csv
```

**Potentially More Efficient**

```text
sales/
  part-001.parquet
  part-002.parquet
  part-003.parquet
```

### Recommended Approach

The lecture mentions files larger than approximately 128 MB as an example of an efficient target size.

This is a useful guideline rather than a universal requirement.

The ideal file size depends on the dataset, format, compression, and workload.

### Important Concept

**Avoid excessive small files when designing Athena datasets.**

---

# 20. Athena Federated Query

Amazon Athena is commonly used to query data stored in Amazon S3.

However, Athena Federated Query extends this capability to supported external data sources.

### What Is Federated Query?

Federated query allows a query engine to access data from multiple systems without first copying all of that data into one central database.

### Architecture

```text
                     AMAZON ATHENA
                           |
                           v
                     SQL QUERY ENGINE
                           |
              +------------+------------+
              |            |            |
              v            v            v
           AMAZON S3    CONNECTOR     CONNECTOR
                           |            |
                           v            v
                        DynamoDB     Amazon RDS
```

### Important Concept

**Athena Federated Query enables SQL queries against supported data sources beyond Amazon S3.**

---

# 21. Athena Federated Query Connectors

The lecture explains that Athena uses data source connectors for federated queries.

These connectors can use AWS Lambda functions to access external data sources.

### Sources Mentioned in the Lecture

- Amazon CloudWatch
- Amazon DynamoDB
- Amazon DocumentDB
- Amazon RDS

### Architecture

```text
                       AMAZON ATHENA
                             |
                             v
                         SQL QUERY
                             |
                             v
                    DATA SOURCE CONNECTOR
                             |
                             v
                        AWS LAMBDA
                             |
                +------------+------------+
                |            |            |
                v            v            v
             DynamoDB     DocumentDB    Amazon RDS
```

### Example

A company stores operational data in DynamoDB.

An analyst wants to query that data using SQL.

An Athena federated query connector can provide access to the supported DynamoDB data source.

### Important Clarification

Connector availability, execution architecture, permissions, and supported features depend on the connector and Athena configuration.

### AWS Exam Point

**Athena Federated Query = Query supported external data sources using connectors.**

---

# 22. AWS Glue vs Amazon Athena

These two services are commonly used together.

| Feature | AWS Glue | Amazon Athena |
|---|---|---|
| Primary Purpose | Data integration and ETL | SQL analytics |
| Data Discovery | Glue Crawlers | Uses registered metadata |
| Metadata Catalog | AWS Glue Data Catalog | Can use Glue Data Catalog |
| Data Transformation | Core capability | SQL-based operations supported |
| Querying S3 | Not its primary purpose | Core capability |
| Apache Spark | Supported for ETL | Not the standard Athena SQL execution engine |
| Typical Use Case | Prepare and transform datasets | Analyze datasets using SQL |

### Architecture

```text
                    RAW DATA IN S3
                          |
                          v
                     AWS GLUE
                          |
                          v
                    CLEAN / TRANSFORM
                          |
                          v
                   OPTIMIZED DATA IN S3
                          |
                          v
                    AMAZON ATHENA
                          |
                          v
                       SQL QUERY
```

### Key Difference

**Glue transforms and integrates data.**

**Athena queries and analyzes data.**

---

# 23. Amazon Athena vs Amazon Redshift

Amazon Athena and Amazon Redshift are both used for analytics, but their architectures differ.

| Feature | Amazon Athena | Amazon Redshift |
|---|---|---|
| Primary Purpose | Interactive SQL queries | Data warehousing and analytics |
| Serverless Option | Yes | Yes, with Redshift Serverless |
| Traditional Data Loading | Not required for S3 queries | Common for warehouse-managed tables |
| S3 Querying | Core capability | Supported through features such as Redshift Spectrum |
| Typical Workload | Ad hoc analytics | Repeated and complex warehouse analytics |
| Infrastructure Management | Minimal | Depends on deployment model |
| Pricing | Depends on selected Athena model | Depends on Redshift deployment and usage |

### When to Choose Athena

Use Athena when:

- Data is already stored in S3.
- You need ad hoc SQL queries.
- You want to avoid a separate data-loading process.
- You want serverless analytics.

### When to Choose Redshift

Consider Redshift when:

- You need a dedicated analytical warehouse.
- You run frequent or complex analytical workloads.
- You require warehouse-oriented performance and data management features.

### Important Concept

Athena and Redshift are not mutually exclusive.

Organizations may use both services for different analytical requirements.

---

# 24. Real-World Example: Application Log Analytics

A company stores application logs in Amazon S3.

The operations team wants to identify errors and investigate incidents.

### Architecture

```text
                    APPLICATION SERVERS
                            |
                            v
                     APPLICATION LOGS
                            |
                            v
                    AMAZON DATA FIREHOSE
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
                       AMAZON ATHENA
                            |
                            v
                         SQL QUERY
```

### Example Query

```sql
SELECT
    error_code,
    COUNT(*) AS error_count
FROM application_logs
WHERE log_level = 'ERROR'
GROUP BY error_code
ORDER BY error_count DESC;
```

### Benefits

- Centralized log analysis
- SQL-based troubleshooting
- No traditional database loading
- Integration with an S3-based data lake

---

# 25. Real-World Example: E-Commerce Sales Analytics

An e-commerce company stores sales transactions in S3.

The business team wants to analyze revenue by city.

### Architecture

```text
                   E-COMMERCE APPLICATION
                            |
                            v
                       SALES RECORDS
                            |
                            v
                         AMAZON S3
                            |
                            v
                     GLUE DATA CATALOG
                            |
                            v
                       AMAZON ATHENA
                            |
                            v
                       SALES ANALYSIS
```

### Example Query

```sql
SELECT
    city,
    SUM(amount) AS total_revenue
FROM sales
GROUP BY city
ORDER BY total_revenue DESC;
```

### Benefits

- Ad hoc business analytics
- Reduced data duplication
- SQL-based reporting
- Serverless query execution

---

# 26. Real-World Example: Optimized Data Lake Analytics

A company stores raw CSV data in Amazon S3.

The dataset is large, and Athena queries are becoming expensive.

The organization wants to optimize the architecture.

### Architecture

```text
                      RAW CSV DATA
                           |
                           v
                        AMAZON S3
                           |
                           v
                       AWS GLUE
                           |
                           v
                 CONVERT CSV TO PARQUET
                           |
                           v
                  PARTITION DATA BY DATE
                           |
                           v
                        AMAZON S3
                           |
                           v
                      AMAZON ATHENA
                           |
                           v
                    OPTIMIZED QUERIES
```

### Benefits

- Reduced scanned data
- Improved query performance
- Lower query costs
- Better data organization

---

# 27. Common Mistakes

### Mistake 1: Assuming Athena Requires Data to Be Loaded into a Database

Athena can query data directly in S3.

### Mistake 2: Assuming Athena Stores the Original Dataset

The original dataset remains in S3.

### Mistake 3: Confusing Schema-on-Read with Schema-on-Write

Athena applies table metadata when reading the dataset.

### Mistake 4: Assuming Athena Requires a Glue Crawler

A crawler is optional.

Metadata can also be created manually or programmatically.

### Mistake 5: Assuming Every Query Reads the Entire Dataset

Partition pruning and columnar formats can reduce the data scanned.

### Mistake 6: Using Excessive Small Files

Too many small files can increase query overhead.

### Mistake 7: Ignoring Columnar Formats

Parquet and ORC can improve query efficiency.

### Mistake 8: Assuming Athena Only Supports S3

Athena Federated Query can access supported external sources.

### Mistake 9: Confusing Athena with AWS Glue

Glue primarily prepares and integrates data.

Athena primarily queries and analyzes data.

### Mistake 10: Assuming Athena Queries Never Modify S3 Data

Ordinary SELECT queries do not modify the source data.

However, Athena also supports operations that can create, write, or modify data, depending on the table type and SQL operation.

---

# 28. AWS Certification Exam Scenarios

## Scenario 1: Query S3 Data Using SQL

A company stores CSV files in Amazon S3.

The company wants to execute SQL queries without loading the data into a database.

**Question:** Which AWS service should be used?

**Answer:** Amazon Athena.

---

## Scenario 2: Automatic Schema Discovery

A company stores thousands of JSON files in S3.

The company wants to automatically discover the data structure before querying it.

**Question:** Which AWS service should be used?

**Answer:** AWS Glue Crawler.

**Reason:** Crawlers discover schemas and populate the Glue Data Catalog.

---

## Scenario 3: Reduce Athena Query Costs

A company frequently queries sales data for a specific month.

The dataset contains multiple years of records.

**Question:** How can query costs be reduced?

**Answer:** Partition the data by appropriate date fields.

**Reason:** Partition pruning reduces unnecessary scanning.

---

## Scenario 4: Optimize Analytical Queries

A company stores large CSV datasets in S3.

Athena queries scan substantial amounts of data.

**Question:** Which file format should the company consider?

**Answer:** Apache Parquet or ORC.

**Reason:** Columnar formats can reduce scanned data.

---

## Scenario 5: Convert CSV to Parquet

A company wants to convert raw CSV datasets into Parquet before running Athena queries.

**Question:** Which AWS service can perform the transformation?

**Answer:** AWS Glue.

---

## Scenario 6: Query DynamoDB Using Athena

A company wants to query supported DynamoDB data using SQL through Athena.

**Question:** Which feature should be used?

**Answer:** Athena Federated Query with an appropriate connector.

---

## Scenario 7: Query Data Without Managing Servers

A company needs an interactive SQL query service for data stored in S3.

The company wants to avoid managing query servers.

**Question:** Which service is suitable?

**Answer:** Amazon Athena.

---

## Scenario 8: Query Multiple Data Sources

A company needs SQL-based analysis across S3 and supported external data sources.

**Question:** Which Athena capability should it consider?

**Answer:** Athena Federated Query.

---

## Scenario 9: Avoid Excessive Small Files

A company stores millions of tiny objects in S3.

Athena queries are experiencing overhead.

**Question:** What optimization should the company consider?

**Answer:** Consolidate small files into appropriately sized files.

---

## Scenario 10: Centralized Table Metadata

A company needs to maintain schema definitions for datasets stored in S3.

**Question:** Which service provides the centralized metadata repository?

**Answer:** AWS Glue Data Catalog.

---

# 29. Interview Questions

### Q1. What is Amazon Athena?

Amazon Athena is a serverless interactive query service that allows users to analyze data in Amazon S3 using SQL.

### Q2. Does Athena require a traditional database?

No. Athena can query supported datasets directly in S3.

### Q3. What is schema-on-read?

A data access approach in which a defined schema is applied when data is read.

### Q4. What is an external table?

A table definition describing data stored outside the query engine.

### Q5. Where is the actual S3 dataset stored?

In Amazon S3.

### Q6. What is the role of AWS Glue Data Catalog?

It stores metadata describing datasets, including schemas and locations.

### Q7. What is the role of a Glue crawler?

It discovers dataset schemas and populates metadata.

### Q8. Is a Glue crawler mandatory for Athena?

No.

### Q9. Which file formats are mentioned in the lecture?

CSV, TSV, JSON, Parquet, and ORC.

### Q10. Why are Parquet and ORC recommended?

They are columnar formats that can reduce data scanning.

### Q11. What is partitioning?

Organizing data into logical groups based on selected partition columns.

### Q12. How does partitioning improve Athena performance?

It enables partition pruning so unrelated data can be skipped.

### Q13. Why should excessive small files be avoided?

They can increase query execution overhead.

### Q14. Can Athena execute JOIN operations?

Yes.

### Q15. Can Athena perform aggregations?

Yes, using functions such as SUM, COUNT, and AVG.

### Q16. What is Athena Federated Query?

A feature that enables Athena to query supported external data sources through connectors.

### Q17. Which AWS service is commonly used to run federated connectors?

AWS Lambda, for Lambda-based connectors.

### Q18. Can Athena query DynamoDB?

Yes, using a supported federated query connector.

### Q19. What is the difference between Athena and Glue?

Athena focuses on SQL querying and analytics, while Glue focuses on data integration and ETL.

### Q20. What is the difference between Athena and Redshift?

Athena is commonly used for serverless SQL analysis of S3 datasets, while Redshift is a managed analytical data warehouse.

### Q21. How can Athena query costs be optimized?

By partitioning datasets, using columnar formats, compressing data, selecting required columns, and avoiding excessive small files.

### Q22. Does a SELECT query modify the original S3 dataset?

No. A normal SELECT query reads the data without changing the source objects.

---

# 30. Key Takeaways

- Amazon Athena is a serverless SQL query service.
- It can query data directly in Amazon S3.
- It does not require loading data into a traditional database.
- Athena uses schema-on-read.
- External tables describe the structure and location of datasets.
- The actual source data remains in S3.
- Athena integrates with AWS Glue Data Catalog.
- AWS Glue Crawlers can discover schemas automatically.
- Athena supports CSV, TSV, JSON, Parquet, and ORC.
- Athena supports SQL filtering, aggregation, sorting, and joins.
- Athena commonly uses usage-based query pricing.
- Reducing data scans can reduce query costs.
- Partitioning improves query efficiency.
- Parquet and ORC support efficient columnar queries.
- Compression can reduce storage and scanning overhead.
- Excessive small files should be avoided.
- AWS Glue can convert raw data into optimized formats.
- Athena Federated Query supports additional data sources.
- Lambda-based connectors can integrate supported external systems.
- Athena is particularly useful for ad hoc analytics and S3-based data lakes.

---

# 31. The Most Important Concepts to Remember

```text
                       AMAZON ATHENA
                       =============

                         DATA SOURCES
                              |
               +--------------+--------------+
               |              |              |
               v              v              v
              CSV            JSON          PARQUET
               |              |              |
               +--------------+--------------+
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
                         TABLE METADATA
                              |
                              v
                         AMAZON ATHENA
                              |
                              v
                           SQL QUERY
                              |
                              v
                         QUERY RESULTS


                    PERFORMANCE OPTIMIZATION
                    ========================

                       Partition Data
                              +
                      Use Parquet / ORC
                              +
                        Compress Data
                              +
                     Avoid Tiny Files
                              |
                              v
                       LESS DATA SCANNED
                              |
                              v
                     LOWER QUERY COSTS


                     ATHENA FEDERATED QUERY
                     ======================

                         AMAZON ATHENA
                              |
                              v
                      FEDERATED CONNECTOR
                              |
                              v
                          AWS LAMBDA
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
              DynamoDB    DocumentDB    Amazon RDS
```

### AWS Exam Memory Points

| Concept | Remember |
|---|---|
| Amazon Athena | Serverless SQL query service |
| Primary Data Source | Amazon S3 |
| Schema-on-Read | Apply schema when reading data |
| External Table | Metadata describing external data |
| Glue Crawler | Automatic schema discovery |
| Glue Data Catalog | Centralized metadata repository |
| SQL | Query language |
| CSV / JSON | Common source formats |
| Parquet / ORC | Columnar formats |
| Partitioning | Reduce unnecessary scans |
| Compression | Reduce data size |
| File Optimization | Avoid excessive small files |
| Federated Query | Query supported external sources |
| Lambda Connector | Access supported external systems |
| AWS Glue | Data preparation and ETL |
| Amazon Redshift | Analytical data warehouse |

---

# 32. Related Topics

- Amazon Athena
- AWS Glue
- AWS Glue Data Catalog
- AWS Glue Crawlers
- Schema-on-Read
- Schema-on-Write
- External Tables
- Amazon S3
- SQL Analytics
- Apache Parquet
- Apache ORC
- Data Partitioning
- Query Optimization
- Athena Federated Query
- AWS Lambda
- Amazon DynamoDB
- Amazon DocumentDB
- Amazon RDS
- Amazon Redshift
- Data Lakes
- AWS Data Analytics
