🐝 Hive Telecom Data Analytics
Apache Hive data warehouse for telecommunication analytics — optimized with partitioning and bucketing to analyze call durations, data usage, and SMS trends across millions of records.

📋 Table of Contents
Problem Statement
Solution Overview
Repository Structure
Schema Reference
Setup & Installation
How to Run Queries
Query Examples
Partitioning & Bucketing Strategy


Problem Statement
A telecommunication company processes millions of records related to customer calls, SMS usage, and data consumption. Without optimization, querying this data for business insights is slow and resource-intensive.
Key business questions this project answers:
#
Business Question
Table
1
What are the total call durations by region and date?
call_data
2
Who are the top data consumers in each region?
data_usage
3
What are the SMS usage trends by region and customer?
sms_data


Solution Overview
This repository uses Hive Partitioning and Bucketing to dramatically improve query performance:
Partitioning — Tables are partitioned by date and region, so Hive only scans the relevant data slice instead of the full dataset.
Bucketing — All tables are bucketed by customer_id into 10 buckets, enabling fast customer-centric lookups and efficient joins.

Repository Structure
hive-telecom/
│
├── ddl/                          # Table creation scripts
│   ├── call_data.hql             # Call data table (partitioned + bucketed)
│   ├── data_usage.hql            # Data usage table (partitioned + bucketed)
│   └── sms_data.hql              # SMS data table (partitioned + bucketed)
│
├── data/                         # Sample data files for local testing
│   ├── sample_call_data.csv
│   ├── sample_data_usage.csv
│   └── sample_sms_data.csv
│
├── queries/                      # Optimized HiveQL query templates
│   ├── call_duration_by_region.hql
│   ├── top_data_users.hql
│   └── sms_usage_trends.hql
│
├── load/                         # Data loading scripts
│   ├── load_call_data.hql
│   ├── load_data_usage.hql
│   └── load_sms_data.hql
│
└── README.md







Schema Reference
call_data
Stores records of all customer calls.
Column
Type
Notes
call_id
INT
Primary key
customer_id
INT
Bucket key
call_duration
FLOAT
Duration in minutes
call_date
STRING
Partition key
region
STRING
Partition key (North / South / East / West)


data_usage
Stores records of customer mobile data consumption.
Column
Type
Notes
usage_id
INT
Primary key
customer_id
INT
Bucket key
data_used
FLOAT
Data consumed in GB
usage_date
STRING
Partition key
region
STRING
Partition key







sms_data
Stores records of customer SMS activity.
Column
Type
Notes
sms_id
INT
Primary key
customer_id
INT
Bucket key
sms_count
INT
Number of SMS sent
sms_date
STRING
Partition key
region
STRING
Partition key


Setup & Installation
Prerequisites
Tool
Version
Apache Hive
3.1+
Apache Spark / PySpark
3.3+
Hadoop / HDFS
3.x
AWS CLI
2.x (for EMR/S3 deployments)
Python
3.8+


hive -f load/load_call_data.hql
hive -f load/load_data_usage.hql
hive -f load/load_sms_data.hql


How to Run Queries
Option A — Hive CLI
hive -f queries/call_duration_by_region.hql
hive -f queries/top_data_users.hql
hive -f queries/sms_usage_trends.hql


Query Examples
Call Durations by Region and Date
SELECT SUM(call_duration) AS total_duration
FROM call_data
WHERE call_date = '2023-08-01'
  AND region = 'North';

✅ Partition pruning on call_date and region means only the matching partition is scanned — not the full table.

Top Data Users by Region
SELECT customer_id, SUM(data_used) AS total_data
FROM data_usage
WHERE region = 'North'
GROUP BY customer_id
ORDER BY total_data DESC;


SMS Usage Trends by Region
SELECT customer_id, SUM(sms_count) AS total_sms
FROM sms_data
WHERE region = 'North'
GROUP BY customer_id;






Partitioning & Bucketing Strategy
Why Partitioning?
Partitioning by date and region ensures that Hive only reads the relevant folder on HDFS/S3 instead of scanning every record. For a dataset with millions of rows spanning multiple years and regions, this reduces scan time dramatically.
/warehouse/call_data/
  call_date=2023-08-01/region=North/
  call_date=2023-08-01/region=South/
  call_date=2023-08-02/region=North/
  ...

Why Bucketing?
Bucketing on customer_id into 10 buckets pre-sorts and distributes customer records across fixed files. This speeds up customer-level aggregations, JOIN operations between call_data, data_usage, and sms_data on customer_id, and sampling queries.
DDL Example — call_data
CREATE TABLE call_data (
    call_id       INT,
    customer_id   INT,
    call_duration FLOAT
)
PARTITIONED BY (call_date STRING, region STRING)
CLUSTERED BY (customer_id) INTO 10 BUCKETS
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/data/test/txt';


