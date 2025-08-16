# 2. Best Practices
## Conduct a POC for Amazon Redshift
In general, we recommend using two weeks of data for an Amazon Redshift POC.

The first time you use Redshift Serverless, the console leads you through the steps required to launch your warehouse. You might also be eligible for a credit towards your Redshift Serverless usage in your account.

For quick ingestion and analysis, you can use Amazon Redshift query editor v2, to easily load data files from your local desktop.

Autocopy (in preview) is an extension of the COPY command and automates continuous data loading from Amazon S3 buckets.

## designing tables - Choose the best sort key
When you use automatic table optimization (default), you don't need to choose the sort key of your table, it is default to AUTO. 

- If recent data is queried most frequently, specify the timestamp column as the leading column for the sort key. 
- If you do frequent range filtering or equality filtering on one column, specify that column as the sort key.
- If you frequently join a table, specify the join column as both the sort key and the distribution key. So the query optimizer will choose a sort merge join, instead of a slower hash join, and bypass the sort phase of the sort merge join.

## designing tables - Choose the best distribution style
Determines how data is distributed across the node slices in your Redshift cluster. The goal is to minimize data movement during joins and aggregations.

When you use automatic table optimization (default), you don't need to choose the distribution style of your table. It will be set to AUTO.

1. Distribute the fact table, and one most commonly joined and large dimension table, on their common columns. Your fact table can have only one distribution key. Any tables that join on another key aren't collocated with the fact table. Designate both the dimension table's PK and the fact table's corresponding FK as the DISTKEY. 
2. Choose the largest dimension based on the size of the filtered dataset (???). 
3. Choose a column with high cardinality in the filtered result set.
4. Change some dimension tables to use ALL distribution. If a dimension table cannot be collocated with the fact table, or other important joining tables, you can improve query performance significantly allowing all the nodes to have a copy of it. Using ALL distribution multiplies storage consumption, and increases load times etc.

## designing tables - Let COPY choose compression encodings
You can specify compression encodings when you create a table, but in most cases, automatic compression produces the best results.

ENCODE AUTO is the default for tables. When a table is set to ENCODE AUTO, Redshift automatically manages compression encoding for all columns in the table.

There is a performance cost for automatic compression encoding, but only if the table is empty and does not already have compression encoding. 

For short-lived tables, and tables that you create frequently, load the table once with automatic compression, or run the ANALYZE COMPRESSION command. Then use those encodings to create new tables. You can add the encodings to the CREATE TABLE statement, or use CREATE TABLE LIKE to create a new table with the same encoding.

## designing tables - Define primary key and foreign key constraints
Define PK and FK constraints between tables wherever appropriate. Even though they are informational only, the query optimizer uses those constraints to generate more efficient query plans.

Redshift does not enforce unique, PK, and FK constraints.

## designing tables - Use the smallest possible column size
Don't use the maximum column size for convenience. 

Instead, define the size based on the the largest values you are likely to store in your columns.

## designing tables - Use date/time data types for date columns
Redshift stores DATE and TIMESTAMP data more efficiently than CHAR or VARCHAR, which results in better query performance. 

## loading data - Loading data files
The COPY command loads data in parallel from Amazon S3, Amazon EMR, Amazon DynamoDB, or multiple data sources on remote hosts. COPY loads large amounts of data much more efficiently than using INSERT statements, and stores data more effectively.

When you load data from a file that can't be split, Redshift is forced to perform a slow serial load. 

These files can be automatically split during data loading:
- an uncompressed CSV file
- a columnar file (Parquet/ORC)

Amazon Redshift automatically splits files 128MB or larger into chunks.

File types such as JSON, or CSV, when compressed, such as GZIP, aren't automatically split. For these we recommend manually splitting the data into multiple smaller files that are close in size, from 1 MB to 1 GB after compression. Additionally, make the number of files a multiple of the number of slices in your cluster.

## loading data - Compressing your data files
Use gzip, lzop, bzip2, or Zstandard to compress the splitted files.

## loading data - Verify data files before and after a load
After the load operation is complete, query the STL_LOAD_COMMITS system table to verify that the expected files were loaded.

## loading data - Use a multi-row insert
If a COPY command is not an option, and you require SQL inserts, use a multi-row insert whenever possible, as data compression more efficient with bulk inserts.

## loading data - Use a bulk insert
Use a bulk insert operation with a SELECT clause for high-performance data insertion.

Use the INSERT and CREATE TABLE AS commands when you need to move data or a subset of data from one table into another.

## loading data - Load data in sort key order
Load your data in sort key order, to avoid needing to vacuum.

## loading data - Use time-series tables
If your data has a fixed retention period, you can organize your data as a sequence of time-series tables. In such a sequence, each table is identical but contains data for different time ranges.

You can remove old data by running a DROP TABLE command on the corresponding tables. This approach is much faster than running a large-scale DELETE process, and saves you from having to run a subsequent VACUUM process to reclaim space. 

To hide the fact that the data is stored in different tables, you can create a UNION ALL view. When you delete old data, refine your UNION ALL view to remove the dropped tables.

To signal the optimizer to avoid scanning tables out of your filtered date range, your view definition need to include filters for the date range that corresponds to each table's date range.

## loading data - Schedule around maintenance windows
If a scheduled maintenance occurs while a query is running, the query is terminated and rolled back, and you need to restart it. 

Schedule long-running operations, such as large data loads or VACUUM operation, to avoid maintenance windows. 

## designing queries
- Avoid using select *. Include only the columns you need.
- Use subqueries (`where my_col in (select my_col from small_table where ...)`) in cases where one table in the query is used only for predicate conditions and the subquery returns a small number of rows (less than about 200). To avoid a join. 
- Avoid using functions in query predicates (`where func(my_col) = '...'`). They can drive up the cost of the query by requiring large numbers of rows to resolve the intermediate steps of the query.
- If you use both GROUP BY and ORDER BY clauses, put the columns in the same order in both. 

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

Use the column ENCODE parameter when creating any tables that you load using the COPY command. 

The best solution is to use column encoding during table creation, because this also maintains the benefit of storing compressed data on disk. 

Ensure that all tables of significant size created during your ETL processes (for example, staging tables and temporary tables) declare a compression encoding for all columns except the first sort key. 

If you are confident that the table will remain extremely small, turn off compression altogether with the COMPUPDATE OFF parameter. Otherwise, create the table with explicit compression before loading it with the COPY command. 

1. Optimize COPY commands that load fewer files than the number of cluster nodes.  
2. Optimize COPY commands that load fewer files than the number of cluster slices.  
3. Optimize COPY commands where the number of files is not a multiple of the number of cluster  slices. 

The number of slices in a node depends on the node size of the cluster.

Amazon Redshift doesn't take file size into account when dividing the workload. Split your load  data files so that the files are about equal size, between 1 MB and 1 GB after compression. 

The cost estimates are based on table statistics gathered using the ANALYZE command.  When statistics are out of date or missing, the database might choose a less efficient plan for query  execution, especially for complex queries. Maintaining current statistics helps complex queries run  in the shortest possible time. 

We recommend  running ANALYZE whenever a significant number of new data rows are loaded into an existing  table with COPY or INSERT commands.

The default ANALYZE threshold is 10%. This default means that the ANALYZE command  skips a given table if fewer than 10% of the table's rows have changed since the last  ANALYZE. As a result, you might choose to issue ANALYZE commands at the end of each ETL  process. Taking this approach means that ANALYZE is often skipped but also ensures that ANALYZE  runs when needed. 

By default, ANALYZE  collects statistics for all columns in the table specified. If needed, you can reduce the time required  to run ANALYZE by running ANALYZE only for the columns where it has the most impact.

You can also let Amazon  Redshift choose which columns to analyze by specifying ANALYZE PREDICATE COLUMNS. 

Short query acceleration (SQA) prioritizes selected short-running queries ahead of longer-running  queries. SQA runs short-running queries in a dedicated space, so that SQA queries aren't forced to  wait in queues behind longer queries. SQA only prioritizes queries that are short-running and are in  a user-defined queue.

An appropriate DISTKEY places a similar number of rows on each node slice and is frequently  referenced in join conditions. An optimized join occurs when tables are joined on their DISTKEY  columns, accelerating query performance. 

Redistributing a large table with ALTER TABLE consumes cluster resources and requires temporary  table locks at various times. Implement each recommendation group when other cluster workload  is light. For more details on optimizing table distribution properties. 

In practice, compound sort keys are more effective than interleaved sort  keys for the vast majority of Amazon Redshift workloads.

If a table is small, it's more  efficient not to have a sort key to avoid sort key storage overhead. 

Changing column compression encodings with ALTER TABLE consumes cluster resources and  requires table locks at various times. It's best to implement recommendations when the cluster  workload is light. 
