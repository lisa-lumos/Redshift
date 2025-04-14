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


































