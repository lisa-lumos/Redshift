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




































