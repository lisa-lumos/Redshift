# 2. Best Practices
## Conduct a proof of concept (POC) for Amazon Redshift
In general, we recommend using two weeks of data for an Amazon Redshift POC.

The first time you use Redshift Serverless, the console leads you through the steps required to launch your warehouse. You might also be eligible for a credit towards your Redshift Serverless usage in your account.

For quick ingestion and analysis, you can use Amazon Redshift query editor v2 to easily load data files from your local desktop.

Autocopy (in preview) is an extension of the COPY command and automates continuous data loading from Amazon S3 buckets.

## designing tables - Choose the best sort key
When you use automatic table optimization, you don't need to choose the sort key of your table.

Some suggestions for the best approach follow:
- To have Amazon Redshift choose the appropriate sort order, specify AUTO for the sort key. 
- If recent data is queried most frequently, specify the timestamp column as the leading column for the sort key. 
- If you do frequent range filtering or equality filtering on one column, specify that column as the sort key.
- If you frequently join a table, specify the join column as both the sort key and the distribution key. Doing this enables the query optimizer to choose a sort merge join instead of a slower hash join. Because the data is already sorted on the join key, the query optimizer can bypass the sort phase of the sort merge join.

## designing tables - Choose the best distribution style
Determines how data is distributed across the slices (or nodes) in your Redshift cluster. The goal is to minimize data movement during joins and aggregations.

When you use automatic table optimization, you don't need to choose the distribution style of your table. 

Some suggestions for the best approach follow:
1. Distribute the fact table and one dimension table on their common columns. Your fact table can have only one distribution key. Any tables that join on another key aren't collocated with the fact table. Choose one dimension to collocate based on how frequently it is joined, and the size of the joining rows. Designate both the dimension table's primary key and the fact table's corresponding foreign key as the DISTKEY. 
2. Choose the largest dimension based on the size of the filtered dataset. Only the rows that are used in the join must be distributed, so consider the size of the dataset after filtering, not the size of the table. 
3. Choose a column with high cardinality in the filtered result set. If you distribute a sales table on a date column, you should probably get fairly even data distribution, unless most of your sales are seasonal. However, if you commonly use a range-restricted predicate to filter for a narrow date period, most of the filtered rows occur on a limited set of slices and the query workload is skewed. 
4. Change some dimension tables to use ALL distribution. If a dimension table cannot be collocated with the fact table or other important joining tables, you can improve query performance significantly by distributing the entire table to all of the nodes. Using ALL distribution multiplies storage space requirements and increases load times and maintenance operations, so you should weigh all factors before choosing ALL distribution.

To have Amazon Redshift choose the appropriate distribution style, specify AUTO for the distribution style.

## designing tables - Let COPY choose compression encodings
You can specify compression encodings when you create a table, but in most cases, automatic compression produces the best results.

ENCODE AUTO is the default for tables. When a table is set to ENCODE AUTO, Amazon Redshift automatically manages compression encoding for all columns in the table.

There is a performance cost for automatic compression encoding, but only if the table is empty and does not already have compression encoding. 

For short-lived tables and tables that you create frequently, such as staging tables, load the table once with automatic compression or run the ANALYZE COMPRESSION command. Then use those encodings to create new tables. You can add the encodings to the CREATE TABLE statement, or use CREATE TABLE LIKE to create a new table with the same encoding.

## designing tables - Define primary key and foreign key constraints
Define primary key and foreign key constraints between tables wherever appropriate. Even though they are informational only, the query optimizer uses those constraints to generate more efficient query plans.

Amazon Redshift does not enforce unique, primary-key, and foreign-key constraints.

## designing tables - Use the smallest possible column size
Don't make it a practice to use the maximum column size for convenience. 

Instead, consider the largest values you are likely to store in your columns and size them accordingly. For instance, a CHAR column for storing U.S. state and territory abbreviations used by the post office only needs to be CHAR(2).

## designing tables - Use date/time data types for date columns
Amazon Redshift stores DATE and TIMESTAMP data more efficiently than CHAR or VARCHAR, which results in better query performance. 

Use the DATE or TIMESTAMP data type, depending on the resolution you need, rather than a character type when storing date/time information.

## loading data - Loading data files
The COPY command loads data in parallel from Amazon S3, Amazon EMR, Amazon DynamoDB, or multiple data sources on remote hosts. COPY loads large amounts of data much more efficiently than using INSERT statements, and stores the data more effectively as well.

When you load data from a file that can't be split, Amazon Redshift is forced to perform a serialized load, which is much slower. 

The following files can be automatically split when their data is loaded:
- an uncompressed CSV file
- a columnar file (Parquet/ORC)

Amazon Redshift automatically splits files 128MB or larger into chunks.

File types such as JSON, or CSV, when compressed with other compression algorithms, such as GZIP, aren't automatically split. For these we recommend manually splitting the data into multiple smaller files that are close in size, from 1 MB to 1 GB after compression. Additionally, make the number of files a multiple of the number of slices in your cluster.

## loading data - Compressing your data files
When you want to compress large load files, we recommend that you use gzip, lzop, bzip2, or Zstandard to compress them and split the data into multiple smaller files.

## loading data - Verify data files before and after a load
After the load operation is complete, query the STL_LOAD_COMMITS system table to verify that the expected files were loaded.

## loading data - Use a multi-row insert
If a COPY command is not an option and you require SQL inserts, use a multi-row insert whenever possible. Data compression is inefficient when you add data only one row or a few rows at a time.

Multi-row inserts improve performance by batching up a series of inserts.

## loading data - Use a bulk insert
Use a bulk insert operation with a SELECT clause for high-performance data insertion.

Use the INSERT and CREATE TABLE AS commands when you need to move data or a subset of data from one table into another.

## loading data - Load data in sort key order
Load your data in sort key order to avoid needing to vacuum.

## loading data - Use time-series tables
If your data has a fixed retention period, you can organize your data as a sequence of time-series tables. In such a sequence, each table is identical but contains data for different time ranges.

You can remove old data by running a DROP TABLE command on the corresponding tables. This approach is much faster than running a large-scale DELETE process and saves you from having to run a subsequent VACUUM process to reclaim space. To hide the fact that the data is stored in different tables, you can create a UNION ALL view. When you delete old data, refine your UNION ALL view to remove the dropped tables.

To signal the optimizer to skip the scan on tables that don't match the query filter, your view definition filters for the date range that corresponds to each table.

## loading data - Schedule around maintenance windows
If a scheduled maintenance occurs while a query is running, the query is terminated and rolled back, and you need to restart it. 

Schedule long-running operations, such as large data loads or VACUUM operation, to avoid maintenance windows. 

## designing queries
- Avoid using select *. Include only the columns you need.
- Use subqueries in cases where one table in the query is used only for predicate conditions and the subquery returns a small number of rows (less than about 200). To avoid a join. 
- Avoid using functions in query predicates. They can drive up the cost of the query by requiring large numbers of rows to resolve the intermediate steps of the query.
- If you use both GROUP BY and ORDER BY clauses, make sure that you put the columns in the same order in both. 

## Viewing Amazon Redshift Advisor recommendations
You can access Amazon Redshift Advisor recommendations using the Amazon Redshift console, Amazon Redshift API, or AWS CLI. To access recommendations you must have permission redshift:ListRecommendationsattached to your IAM role or identity.

To help you  prioritize your optimizations, Advisor ranks recommendations by order of impact. 

Advisor only displays recommendations that should have a significant impact on performance and operations.

Ensure that each COPY that loads a significant amount of data, or runs for a significant duration, ingests compressed data objects from Amazon S3.

The ideal object size is 1-128 MB after compression. 

As a best practice, we recommend isolating databases in Amazon Redshift from one another. Queries run in a specific database and can't access data from any other database on the cluster. 

Consider moving each actively queried database to a separate dedicated cluster.

Because a user must connect to each database specifically, and queries can only access a single database, moving databases to separate clusters has minimal impact for users. 

Workload  management (WLM) defines how queries are routed to the queues. Amazon Redshift allocates each queue a portion of the cluster's available memory. A queue's memory is divided  among the queue's query slots. 

Consider reducing the configured slot count for queues where the slots have never been fully used.

When you load data into an empty table with compression encoding declared with the COPY command, Amazon Redshift applies storage compression. This optimization ensures that data in your cluster is stored efficiently even when loaded by end users. The analysis required to apply compression can require significant time. 

When you load data as part of a structured process, such as in an overnight extract, transform, load (ETL) batch, you can define the compression beforehand. You can also optimize your table definitions to skip this phase permanently without any negative impacts. 




















