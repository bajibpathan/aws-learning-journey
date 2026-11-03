# AWS Glue and Amazon Athena – Hands-On Lab

## Analyzing Weekly Sales Data Using Serverless AWS Analytics

**Category:** AWS Data Analytics  
**Services:** Amazon S3, AWS Glue, Amazon Athena, AWS IAM  
**Difficulty:** Beginner  
**Lab Type:** Hands-On  
**Region:** Europe (London) – `eu-west-2`

---

# 1. Lab Overview

In this hands-on exercise, we will build a serverless data analytics solution using Amazon S3, AWS Glue, and Amazon Athena.

The objective is to analyze weekly sales data from ten coffee shops across the United Kingdom.

The sales information is provided as a CSV file containing transaction-level sales information.

We will store this dataset in Amazon S3, use AWS Glue to discover its schema, and query the data using Amazon Athena.

### What We Will Build

```text
                  WEEKLY SALES CSV FILE
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
                   DATABASE AND TABLE
                           |
                           v
                      AMAZON ATHENA
                           |
                           v
                        SQL QUERIES
                           |
                           v
                     SALES ANALYSIS
```

### Learning Objectives

By completing this lab, we will learn how to:

1. Create an Amazon S3 bucket.
2. Upload a CSV dataset.
3. Configure an AWS Glue crawler.
4. Create a Glue Data Catalog database and table.
5. Configure Athena query results storage.
6. Query S3 data using SQL.
7. Analyze sales revenue.
8. Identify best-selling products.
9. Analyze daily sales by location.
10. Understand how AWS Glue and Athena work together.

---

# 2. Business Scenario

A coffee shop business operates ten locations across the United Kingdom.

The organization generates weekly sales reports containing information about the products sold at each location.

The business wants to answer three questions:

**Question 1:** Which coffee shop generated the highest revenue?

**Question 2:** Which products sold the most units?

**Question 3:** How did daily sales perform in London?

Instead of loading the CSV dataset into a relational database, we will use Amazon Athena to analyze it directly in Amazon S3.

### Business Requirements

| Requirement | AWS Solution |
|---|---|
| Store weekly sales data | Amazon S3 |
| Discover CSV structure | AWS Glue Crawler |
| Store schema metadata | AWS Glue Data Catalog |
| Execute SQL queries | Amazon Athena |
| Store Athena query results | Amazon S3 |
| Control service permissions | AWS IAM |

---

# 3. Solution Architecture

```text
                      LOCAL COMPUTER
                            |
                            |
                     Weekly Sales CSV
                            |
                            v
                    +----------------+
                    |   AMAZON S3    |
                    |----------------|
                    | sales/         |
                    | weekly.csv     |
                    +----------------+
                            |
                            |
                            v
                    +----------------+
                    |  GLUE CRAWLER  |
                    +----------------+
                            |
                            | Discover Schema
                            v
                    +----------------+
                    | GLUE DATA      |
                    | CATALOG        |
                    |----------------|
                    | Database       |
                    | Sales Table    |
                    | Column Types   |
                    +----------------+
                            |
                            |
                            v
                    +----------------+
                    | AMAZON ATHENA  |
                    |----------------|
                    | SQL Queries    |
                    | Aggregations   |
                    | Filtering      |
                    +----------------+
                            |
                            v
                    +----------------+
                    | QUERY RESULTS  |
                    |----------------|
                    | Revenue        |
                    | Products       |
                    | Daily Sales    |
                    +----------------+
                            |
                            v
                    +----------------+
                    |   AMAZON S3    |
                    |----------------|
                    | athena/        |
                    | Query Outputs  |
                    +----------------+
```

### Architecture Components

| Component | Purpose |
|---|---|
| Local CSV File | Contains weekly sales data |
| Amazon S3 | Stores the original dataset |
| AWS Glue Crawler | Discovers the CSV schema |
| Glue Data Catalog | Stores database and table metadata |
| Amazon Athena | Executes SQL queries |
| Athena Results Location | Stores query results |
| IAM Role | Allows the crawler to access required resources |

---

# 4. Prerequisites

Before starting the lab, ensure that you have:

- An AWS account
- Access to the AWS Management Console
- Permissions to use Amazon S3
- Permissions to use AWS Glue
- Permissions to use Amazon Athena
- Permissions to create or use an appropriate IAM role
- The sample weekly sales CSV file

### Required Dataset

Download the weekly sales CSV file provided in the course resources.

The instructor also provides a SQL file containing sample queries.

**Important:** The exact dataset and SQL file must be obtained from the course resources. They were referenced in the transcript but were not included in the uploaded lecture file.

---

# 5. Understanding the Sales Dataset

The CSV file contains weekly sales information from ten coffee shops.

### Locations

The lecture identifies locations including:

- London
- Manchester
- Birmingham
- Leeds
- Glasgow
- Bristol
- Liverpool
- Edinburgh
- Nottingham
- Cardiff

### Dataset Structure

The lecture describes the following fields:

| Column | Description |
|---|---|
| Shop ID | Coffee shop identifier or name |
| Town/City | Location of the shop |
| Date | Sales date |
| Product | Product sold |
| Category | Product category |
| Quantity Sold | Number of units sold |
| Total Sales | Revenue generated |

### Example Data

The instructor provides an example for a London coffee shop.

| Shop | City | Date | Product | Category | Quantity | Total Sales |
|---|---|---|---|---|---:|---:|
| Crimson Roast | London | 2025-07-18 | Butter Croissant | Pastry | 177 | £531 |
| Crimson Roast | London | 2025-07-18 | Cinnamon Roll | Pastry | 62 | £186 |

These values are examples described in the lecture, not a complete reproduction of the CSV dataset.

### Important Concept

The CSV file contains the actual sales data.

AWS Glue discovers its structure.

Amazon Athena queries the data using the discovered metadata.

---

# 6. Step 1 – Create an Amazon S3 Bucket

The first step is to create an S3 bucket to store the sales dataset.

### AWS Console Navigation

```text
AWS Management Console
        |
        v
Amazon S3
        |
        v
Create Bucket
```

### Configuration

| Setting | Value |
|---|---|
| AWS Region | Europe (London) – eu-west-2 |
| Bucket Type | General purpose |
| Bucket Name | Globally unique name |
| Public Access | Keep blocked |
| Encryption | Use default encryption |

### Example Bucket Name

```text
ritual-roast-weekly-sales-<unique-suffix>
```

S3 bucket names must be globally unique.

Use your own unique suffix.

### Steps

1. Sign in to the AWS Management Console.
2. Select the London Region.
3. Open Amazon S3.
4. Click **Create bucket**.
5. Enter a globally unique bucket name.
6. Keep public access blocked.
7. Retain the default encryption settings.
8. Click **Create bucket**.

### Expected Result

An empty S3 bucket is created successfully.

---

# 7. Step 2 – Create an S3 Folder for Sales Data

Inside the newly created bucket, create a folder named:

```text
sales/
```

### Steps

1. Open the S3 bucket.
2. Click **Create folder**.
3. Enter:

```text
sales
```

4. Click **Create folder**.

### Expected Structure

```text
ritual-roast-weekly-sales-<unique-suffix>/
    |
    +-- sales/
```

### Why Use a Separate Folder?

Organizing source data into dedicated prefixes makes it easier to:

- Locate datasets
- Configure Glue crawler targets
- Separate source data from query results
- Manage access and lifecycle policies

---

# 8. Step 3 – Upload the Weekly Sales CSV File

Upload the sample CSV file into the `sales/` folder.

### Steps

1. Open the S3 bucket.
2. Navigate to the `sales/` folder.
3. Click **Upload**.
4. Click **Add files**.
5. Select the weekly sales CSV file.
6. Click **Upload**.

### Expected Structure

```text
ritual-roast-weekly-sales-<unique-suffix>/
    |
    +-- sales/
          |
          +-- weekly-sales.csv
```

The filename shown here is illustrative; use the actual name of the course dataset.

### Important Concept

Amazon S3 stores the actual CSV data.

AWS Glue and Athena will use this data without requiring it to be loaded into a traditional database.

### Suggested Screenshot

```text
01-s3-sales-dataset.png
```

Capture the uploaded CSV file inside the S3 `sales/` folder.

---

# 9. Step 4 – Create an S3 Folder for Athena Results

Amazon Athena needs a location to store query results when using the standard S3-backed results configuration.

We will create a separate folder in the same S3 bucket.

### Folder Name

```text
athena/
```

### Steps

1. Open the S3 bucket.
2. Click **Create folder**.
3. Enter:

```text
athena
```

4. Click **Create folder**.

### Expected Structure

```text
ritual-roast-weekly-sales-<unique-suffix>/
    |
    +-- sales/
    |     |
    |     +-- weekly-sales.csv
    |
    +-- athena/
```

### Important Concept

The `sales/` prefix stores the source dataset.

The `athena/` prefix stores query output files.

**Do not configure the Glue crawler to scan the Athena query-results folder.**

This avoids mixing query output metadata with the original sales dataset.

---

# 10. Step 5 – Configure the AWS Glue Crawler

The next step is to discover the structure of the CSV dataset.

AWS Glue Crawlers examine supported data sources and populate the AWS Glue Data Catalog.

### Architecture

```text
                    AMAZON S3
                        |
                        v
                   sales/*.csv
                        |
                        v
                   GLUE CRAWLER
                        |
                        v
                  DISCOVER SCHEMA
                        |
                        v
                  GLUE DATA CATALOG
```

### AWS Console Navigation

```text
AWS Management Console
        |
        v
AWS Glue
        |
        v
Crawlers
        |
        v
Create Crawler
```

### Suggested Configuration

| Setting | Example |
|---|---|
| Crawler Name | ritual-roast-sales-crawler |
| Data Source | Amazon S3 |
| S3 Path | s3://your-bucket-name/sales/ |
| IAM Role | Appropriate Glue execution role |
| Target Database | ritual_roast_analytics |
| Schedule | On demand |

### Steps

1. Open AWS Glue.
2. Navigate to **Crawlers**.
3. Click **Create crawler**.
4. Enter the crawler name.
5. Select Amazon S3 as the data source.
6. Specify the S3 `sales/` prefix.
7. Configure an appropriate IAM role.
8. Select or create the target Glue database.
9. Configure the crawler to run on demand.
10. Review the configuration.
11. Create the crawler.

### IAM Requirements

The crawler execution role needs permission to:

- Access the required S3 objects
- Read dataset metadata
- Create or update appropriate Data Catalog metadata
- Write operational logs where required

Use the least privileges necessary.

---

# 11. Step 6 – Run the AWS Glue Crawler

After creating the crawler, execute it.

### Steps

1. Open AWS Glue.
2. Navigate to **Crawlers**.
3. Select the crawler.
4. Click **Run crawler**.
5. Wait for the crawler to complete.

### Expected Result

The crawler discovers the CSV dataset schema and creates or updates table metadata in the Glue Data Catalog.

### Architecture

```text
                 WEEKLY SALES CSV
                        |
                        v
                   GLUE CRAWLER
                        |
                        v
                   SCHEMA DISCOVERY
                        |
                        v
                   GLUE DATABASE
                        |
                        v
                    SALES TABLE
```

### Expected Metadata

The table should describe fields corresponding to the dataset, including:

- Shop ID
- Town/City
- Date
- Product
- Category
- Quantity Sold
- Total Sales

The exact column names and data types depend on the CSV file and crawler inference.

### Suggested Screenshot

```text
02-glue-crawler-success.png
```

Capture the crawler showing a successful completion.

---

# 12. Step 7 – Verify the Glue Data Catalog

Once the crawler finishes, inspect the generated metadata.

### AWS Console Navigation

```text
AWS Glue
    |
    v
Data Catalog
    |
    v
Databases
    |
    v
ritual_roast_analytics
    |
    v
Tables
```

### Expected Result

A database containing a table representing the weekly sales dataset.

### Example

```text
Glue Data Catalog
        |
        v
Database:
ritual_roast_analytics
        |
        v
Table:
sales
        |
        +-- shop_id
        +-- town_city
        +-- date
        +-- product
        +-- category
        +-- quantity_sold
        +-- total_sales
```

### Important Concept

**The Glue Data Catalog contains metadata, not the actual CSV records.**

The CSV file remains in Amazon S3.

### Suggested Screenshot

```text
03-glue-data-catalog-table.png
```

Capture the generated table schema.

---

# 13. Step 8 – Configure Amazon Athena

We will now use Athena to query the CSV dataset.

### AWS Console Navigation

```text
AWS Management Console
        |
        v
Amazon Athena
        |
        v
Query Editor
```

### Configure Query Results

1. Open Amazon Athena.
2. Navigate to the Query Editor.
3. Open the query settings for the selected workgroup.
4. Configure the query result location if required.
5. Specify the S3 `athena/` prefix.
6. Save the settings.

### Example Location

```text
s3://ritual-roast-weekly-sales-<unique-suffix>/athena/
```

### Important Clarification

Athena workgroups can have different query-results configurations.

For this lab, use the standard S3 query-results location described in the lecture.

### Suggested Screenshot

```text
04-athena-query-results-location.png
```

Capture the Athena settings showing the S3 results location.

---

# 14. Step 9 – Verify the Database and Table in Athena

Athena can use the metadata created by the AWS Glue crawler.

### Steps

1. Open Athena Query Editor.
2. Select the appropriate data source or catalog.
3. Select the Glue database.
4. Verify the sales table is available.
5. Expand the table to inspect its columns.

### Expected Structure

```text
Amazon Athena
      |
      v
AWS Glue Data Catalog
      |
      v
Database:
ritual_roast_analytics
      |
      v
Table:
sales
```

### Important Concept

Athena uses the table metadata to interpret the CSV data stored in S3.

It does not need to import the entire dataset into a separate relational database.

### Suggested Screenshot

```text
05-athena-database-and-table.png
```

Capture the database, table, and column definitions visible in Athena.

---

# 15. Step 10 – Query Total Revenue by Shop

The first business requirement is to identify which coffee shop generated the highest revenue.

### Business Question

**Which coffee shop generated the highest total sales during the week?**

### SQL Query

```sql
SELECT
    shop_id,
    SUM(total_sales) AS total_revenue
FROM sales
GROUP BY shop_id
ORDER BY total_revenue DESC;
```

### SQL Explanation

| SQL Clause | Purpose |
|---|---|
| SELECT | Selects the required columns |
| SUM | Calculates total revenue |
| FROM | Specifies the source table |
| GROUP BY | Groups records by shop |
| ORDER BY | Sorts the revenue |
| DESC | Displays highest revenue first |

### Expected Result

The query returns each coffee shop and its total weekly revenue.

According to the lecture, **Coco Infuse** generated the highest weekly revenue.

The complete revenue figures were not provided in the transcript.

### Suggested Screenshot

```text
06-athena-revenue-by-shop.png
```

Capture the SQL query and its results.

---

# 16. Step 11 – Query Best-Selling Products

The second business requirement is to identify which products sold the most units.

### Business Question

**Which products generated the highest sales volume across all ten coffee shops?**

### SQL Query

```sql
SELECT
    product,
    SUM(quantity_sold) AS total_units
FROM sales
GROUP BY product
ORDER BY total_units DESC;
```

### SQL Explanation

| SQL Clause | Purpose |
|---|---|
| SELECT | Retrieves product names |
| SUM | Calculates total units sold |
| FROM | Reads the sales table |
| GROUP BY | Groups records by product |
| ORDER BY | Sorts by total units |
| DESC | Shows best-selling products first |

### Results Mentioned in the Lecture

| Product | Units Sold |
|---|---:|
| Almond Danish | 9,160 |
| Cinnamon Roll | 8,573 |

The instructor reports that Almond Danish was the top-selling product and Cinnamon Roll ranked sixth.

These are results from the course demonstration.

### Suggested Screenshot

```text
07-athena-best-selling-products.png
```

Capture the SQL query and product rankings.

---

# 17. Step 12 – Query Daily Sales in London

The third business requirement is to analyze daily sales performance in London.

### Business Question

**What was the total revenue generated each day by the London coffee shop?**

### SQL Query

```sql
SELECT
    date,
    SUM(total_sales) AS daily_sales
FROM sales
WHERE town_city = 'London'
GROUP BY date
ORDER BY date;
```

### SQL Explanation

| SQL Clause | Purpose |
|---|---|
| SELECT | Retrieves the date |
| SUM | Calculates daily revenue |
| FROM | Specifies the sales table |
| WHERE | Filters records for London |
| GROUP BY | Groups sales by date |
| ORDER BY | Sorts results by date |

### Expected Result

The query returns daily sales totals for London.

According to the lecture, **18 July 2025** was the highest-sales day for the London location during the reporting period.

### Important Note

The lecture indicates that the date field was inferred as a string.

If dates are stored in a sortable ISO format such as `YYYY-MM-DD`, string ordering can work as expected.

For other formats, consider converting the field to a proper date type before performing chronological analysis.

### Suggested Screenshot

```text
08-athena-london-daily-sales.png
```

Capture the SQL query and daily sales results.

---

# 18. Understanding the SQL Operations

This lab demonstrates several common SQL operations.

### SELECT

Retrieves specified columns.

```sql
SELECT shop_id
FROM sales;
```

### SUM

Calculates the total of a numeric column.

```sql
SELECT SUM(total_sales)
FROM sales;
```

### GROUP BY

Groups records based on a selected field.

```sql
SELECT
    shop_id,
    SUM(total_sales)
FROM sales
GROUP BY shop_id;
```

### WHERE

Filters records.

```sql
SELECT *
FROM sales
WHERE town_city = 'London';
```

### ORDER BY

Sorts query results.

```sql
SELECT *
FROM sales
ORDER BY total_sales DESC;
```

### Important Concept

Athena supports SQL operations that make it possible to analyze datasets without first loading them into a traditional database.

---

# 19. How AWS Glue and Athena Work Together

This lab demonstrates a common serverless analytics architecture.

### AWS Glue Responsibilities

- Connect to the data source
- Discover the schema
- Create table metadata
- Store metadata in the Data Catalog

### Amazon Athena Responsibilities

- Access the table metadata
- Interpret the S3 dataset
- Execute SQL queries
- Return analytical results

### Architecture

```text
                    AMAZON S3
                        |
                        v
                   WEEKLY SALES CSV
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
                   EXTERNAL TABLE
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

### Key Distinction

| AWS Service | Responsibility |
|---|---|
| Amazon S3 | Stores the actual dataset |
| AWS Glue Crawler | Discovers the schema |
| Glue Data Catalog | Stores metadata |
| Amazon Athena | Queries the dataset |
| Amazon S3 Results Prefix | Stores query output |

---

# 20. Cost Considerations

This lab uses managed AWS services.

However, charges may still apply.

### Potential Costs

| Service | Potential Charges |
|---|---|
| Amazon S3 | Storage and requests |
| AWS Glue | Crawler execution |
| Amazon Athena | Data scanned or selected capacity model |
| CloudWatch | Logging and related usage, where applicable |

### Cost Optimization Tips

- Use a small sample dataset.
- Run the crawler only when required.
- Avoid unnecessary queries.
- Remove unused S3 objects.
- Delete lab resources after completion.

### Important Concept

Serverless does not mean free.

Always monitor AWS usage and billing.

---

# 21. Troubleshooting

## Issue 1 – Glue Crawler Cannot Access S3

### Possible Cause

The crawler execution role does not have the required S3 permissions.

### Resolution

Verify that the IAM role can access the configured S3 source prefix.

---

## Issue 2 – Glue Table Is Not Created

### Possible Causes

- Incorrect S3 path
- Unsupported or unexpected file structure
- Crawler execution failure
- Insufficient permissions

### Resolution

Check the crawler configuration and execution logs.

---

## Issue 3 – Athena Cannot Find the Table

### Possible Causes

- Wrong AWS Region
- Wrong Glue database selected
- Crawler did not complete successfully
- Insufficient catalog permissions

### Resolution

Verify the database and table in AWS Glue and confirm Athena is using the correct Region and catalog.

---

## Issue 4 – Athena Query Fails

### Possible Causes

- Incorrect column names
- Incorrect table name
- Data type mismatch
- Invalid SQL syntax

### Resolution

Inspect the Glue table schema and adjust the query to use the actual discovered column names.

---

## Issue 5 – Athena Query Results Location Error

### Possible Cause

The selected workgroup does not have a usable query-results configuration.

### Resolution

Verify the S3 results location and required permissions.

---

## Issue 6 – Incorrect Query Results

### Possible Causes

- Incorrect inferred schema
- CSV header interpreted as data
- Incorrect field delimiter
- Numeric values interpreted as strings
- Unexpected date format

### Resolution

Inspect the source CSV and Glue table definition.

---

# 22. Lab Validation Checklist

Use the following checklist before considering the lab complete.

- [ ] AWS Region selected
- [ ] S3 bucket created
- [ ] `sales/` prefix created
- [ ] Weekly sales CSV uploaded
- [ ] `athena/` prefix created
- [ ] Glue crawler configured
- [ ] Glue IAM role configured
- [ ] Glue crawler executed successfully
- [ ] Glue database created
- [ ] Glue table created
- [ ] Table schema verified
- [ ] Athena query-results location configured
- [ ] Glue database visible in Athena
- [ ] Revenue-by-shop query executed
- [ ] Best-selling-products query executed
- [ ] London daily-sales query executed
- [ ] Query results validated
- [ ] Required screenshots captured
- [ ] Lab resources cleaned up

---

# 23. Recommended Screenshots

For GitHub documentation, capture the following screenshots.

| # | Screenshot | Filename |
|---|---|---|
| 1 | CSV file uploaded to S3 | `01-s3-sales-dataset.png` |
| 2 | Glue crawler completed successfully | `02-glue-crawler-success.png` |
| 3 | Glue Data Catalog table schema | `03-glue-data-catalog-table.png` |
| 4 | Athena query-results location | `04-athena-query-results-location.png` |
| 5 | Athena database and table | `05-athena-database-and-table.png` |
| 6 | Revenue by shop | `06-athena-revenue-by-shop.png` |
| 7 | Best-selling products | `07-athena-best-selling-products.png` |
| 8 | London daily sales | `08-athena-london-daily-sales.png` |

### Minimum Screenshots for GitHub

To keep documentation simple, the four most useful screenshots are:

1. S3 dataset
2. Glue Data Catalog table
3. Revenue query results
4. Best-selling products query results

---

# 24. AWS Certification Exam Scenarios

## Scenario 1 – Query S3 Data Without a Database

A company stores sales reports in Amazon S3.

The company wants to execute SQL queries without loading the data into a database.

**Answer:** Amazon Athena.

---

## Scenario 2 – Discover CSV Schema

A company wants to automatically identify the structure of CSV files stored in S3.

**Answer:** AWS Glue Crawler.

---

## Scenario 3 – Store Dataset Metadata

A company needs a centralized repository containing table and column definitions.

**Answer:** AWS Glue Data Catalog.

---

## Scenario 4 – Query Revenue by Shop

A company wants to calculate total revenue for each location.

**Answer:** Use Amazon Athena with SQL aggregation and `GROUP BY`.

---

## Scenario 5 – Analyze Sales for a Specific City

A company wants to analyze sales records for London only.

**Answer:** Use a SQL `WHERE` clause in Athena.

---

## Scenario 6 – Minimize Infrastructure Management

A company wants a serverless solution for analyzing CSV datasets in S3.

**Answer:** Amazon Athena integrated with AWS Glue Data Catalog.

---

## Scenario 7 – Reduce Athena Query Costs

A company stores large CSV datasets and wants to reduce data scanned.

**Answer:** Consider converting data to Parquet, partitioning datasets, and selecting only required columns.

---

## Scenario 8 – Troubleshoot Missing Tables

A Glue crawler completed, but the table is not visible in Athena.

**Answer:** Verify the Region, selected catalog, database, table metadata, and permissions.

---

# 25. Interview Questions

### Q1. What is the objective of this lab?

To analyze sales data stored in Amazon S3 using AWS Glue and Amazon Athena.

### Q2. Why use Amazon S3?

To store the original CSV dataset.

### Q3. Why use an AWS Glue crawler?

To automatically discover the dataset schema.

### Q4. What does the Glue Data Catalog store?

Metadata describing the dataset.

### Q5. Does the Glue Data Catalog store the actual CSV records?

No.

### Q6. Why use Amazon Athena?

To query S3 data using SQL without loading it into a traditional database.

### Q7. What is an external table?

A table definition describing data stored outside the query engine.

### Q8. Where does Athena store query results in this lab?

In the configured S3 `athena/` prefix.

### Q9. Which SQL function calculates total revenue?

`SUM()`.

### Q10. Which SQL clause groups records by shop?

`GROUP BY`.

### Q11. Which SQL clause filters records for London?

`WHERE`.

### Q12. Which SQL clause sorts revenue from highest to lowest?

`ORDER BY ... DESC`.

### Q13. What is schema-on-read?

Applying a defined schema when data is read.

### Q14. What happens if the crawler infers an incorrect data type?

The table metadata may need to be corrected.

### Q15. Why are IAM permissions required?

AWS services need authorization to access the required resources.

### Q16. Does Athena require the CSV file to be loaded into RDS?

No.

### Q17. Can Athena query multiple S3 datasets?

Yes, provided appropriate table definitions and permissions are configured.

### Q18. Why should the crawler target only the sales prefix?

To avoid discovering unrelated files, including Athena query outputs.

### Q19. What is the difference between Glue and Athena?

Glue discovers and integrates data, while Athena executes analytical SQL queries.

### Q20. What is the main architectural benefit of this solution?

It provides serverless SQL analytics over S3 data with minimal infrastructure management.

---

# 26. Key Takeaways

- Amazon S3 stores the actual weekly sales dataset.
- AWS Glue Crawlers discover the dataset structure.
- AWS Glue Data Catalog stores the schema metadata.
- Amazon Athena uses the metadata to query S3 data.
- Athena does not require a traditional relational database.
- SQL can be used to calculate revenue, product sales, and daily performance.
- `SUM()` performs aggregation.
- `GROUP BY` groups records.
- `WHERE` filters records.
- `ORDER BY` sorts results.
- Athena query results can be stored in a dedicated S3 prefix.
- IAM permissions are required for secure service integration.
- Glue and Athena together provide a practical serverless analytics architecture.

---

# 27. Final Architecture

```text
                       WEEKLY SALES DATA
                              |
                              v
                         AMAZON S3
                              |
                              v
                        SALES CSV FILE
                              |
                              v
                        GLUE CRAWLER
                              |
                              v
                      SCHEMA DISCOVERY
                              |
                              v
                     GLUE DATA CATALOG
                              |
                              v
                       DATABASE / TABLE
                              |
                              v
                        AMAZON ATHENA
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
              Revenue       Product      Daily Sales
              by Shop       Rankings      in London
                 |            |            |
                 +------------+------------+
                              |
                              v
                         QUERY RESULTS
                              |
                              v
                           AMAZON S3
```

---

# 28. Lab Cleanup

After completing the exercise, remove resources that are no longer required.

### Cleanup Steps

1. Delete the Glue crawler.
2. Delete the lab table from the Glue Data Catalog.
3. Delete the lab database if it is no longer required.
4. Delete the source CSV file from S3.
5. Delete Athena query-results objects.
6. Delete the S3 bucket if it was created only for this lab.
7. Remove any lab-specific IAM roles that are no longer needed.

### Important

Deleting a Glue table does not automatically delete the underlying S3 dataset.

Likewise, deleting the source data does not necessarily remove its Data Catalog metadata.

Clean up both separately.

---

# 29. Suggested GitHub Project Structure

```text
data-analytics/
│
├── 01-amazon-kinesis-data-streams.md
├── 02-amazon-data-firehose.md
├── 03-aws-glue-fundamentals.md
├── 04-amazon-athena.md
│
└── labs/
    └── 01-aws-glue-and-amazon-athena-hands-on/
        │
        ├── README.md
        │
        ├── sample-queries.sql
        │
        └── screenshots/
            ├── 01-s3-sales-dataset.png
            ├── 02-glue-crawler-success.png
            ├── 03-glue-data-catalog-table.png
            ├── 04-athena-query-results-location.png
            ├── 05-athena-database-and-table.png
            ├── 06-athena-revenue-by-shop.png
            ├── 07-athena-best-selling-products.png
            └── 08-athena-london-daily-sales.png
```
