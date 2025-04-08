# introduction
## Architecture
The core infrastructure component of an Amazon Redshift data warehouse is a cluster.

A cluster is composed of one/more compute nodes. If a cluster is provisioned with two or more compute nodes, an additional leader node coordinates the compute nodes and handles external communication. Your client application interacts directly only with the leader node. The compute nodes are transparent to external applications.

It parses and develops execution plans to carry out database operations, aka, the series of steps necessary to obtain results for complex queries. Based on the execution plan, the leader node compiles code, distributes the compiled code to the compute nodes, and assigns a portion of the data to each compute node.

The leader node distributes SQL statements to the compute nodes only when a query references tables that are stored on the compute nodes. All other queries run exclusively on the leader node.

The compute nodes run the compiled code and send intermediate results back to the leader node for final aggregation.

Data warehouse data is stored in a separate storage tier Redshift Managed Storage (RMS). RMS provides the ability to scale your storage to petabytes using Amazon S3 storage. RMS lets you scale and pay for computing and storage independently, so that you can size your cluster based only on your computing needs.

A compute node is partitioned into slices. Each slice is allocated a portion of the node's memory and disk space, where it processes a portion of the workload assigned to the node. The leader node manages distributing data to the slices and apportions the workload for any queries or other database operations to the slices. The slices then work in parallel to complete the operation.

The number of slices per node is determined by the node size of the cluster.

When you create a table, you can optionally specify one column as the distribution key. When the table is loaded with data, the rows are distributed to the node slices according to the distribution key that is defined for a table. Choosing a good distribution key enables Amazon Redshift to use parallel processing to load data and run queries efficiently.

A cluster contains one or more databases. User data is stored on the compute nodes.

Amazon Redshift is based on PostgreSQL. Amazon Redshift and PostgreSQL have many differences that you need to take into account, as you design and develop your data warehouse applications.

## Performance
By selecting an appropriate distribution key for each table, you can optimize the distribution of data to balance the workload and minimize movement of data from node to node.

Loading data from multiple flat files takes advantage of parallel processing by spreading the workload across multiple nodes while simultaneously reading from multiple files.

Columnar storage.

Data compression.

The best way to enable data compression on table columns is by allowing Amazon Redshift to apply optimal compression encodings when you load the table with data.

Amazon Redshift caches the results of certain types of queries in memory on the leader node. When a user submits a query, Amazon Redshift checks the results cache for a valid, cached copy of the query results. If a match is found in the result cache, Amazon Redshift uses the cached results and doesn't run the query. Result caching is transparent to the user.

Result caching is turned on by default. You can turn it off for a session with a command. 

Amazon Redshift uses cached results for a new query when all of the following are true:
- The user submitting the query has access permission to the objects used in the query.
- The table or views in the query haven't been modified.
- The query doesn't use a function that must be evaluated each time it's run, such as GETDATE.
- The query doesn't reference Amazon Redshift Spectrum external tables.
- Configuration parameters that might affect query results are unchanged.
- The query syntactically matches the cached query.

Amazon Redshift doesn't cache some large query result sets. It determines whether to cache query results based on a number of factors, including the number of entries in the cache, and the instance type of your cluster.

The compiled code is cached and shared across sessions on the same cluster. As a result, future runs of the same query will be faster, often even with different parameters.





































