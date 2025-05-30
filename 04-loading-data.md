# 4. Loading data
If  keeping data on Amazon S3  is not needed, you can often  consider streaming your data  directly into Amazon Redshift. 

A COPY command is the most efficient way to load a table. You can also add data to your tables  using INSERT commands, though it is much less efficient than using COPY.

After your initial data load, if you add, modify, or delete a significant amount of data, you should  follow up by running a VACUUM command to reorganize your data and reclaim space after deletes.  You should also run an ANALYZE command to update table statistics. 

With role-based access control, Amazon Redshift temporarily assumes an AWS Identity and Access  Management (IAM) role on your behalf. Then, based on the authorizations granted to the role,  Amazon Redshift can access the required AWS resources.  We recommend using role-based access control because it is provides more secure, fine-grained  control of access to AWS resources and sensitive user data, in addition to safeguarding your AWS  credentials. 

To use role-based access control, you must first create an IAM role using the Amazon Redshift  service role type, and then attach the role to your data warehouse. The role must have, at a  minimum, the permissions listed in IAM permissions for COPY, UNLOAD, and CREATE LIBRARY.

A deep copy recreates and repopulates a table by using a bulk insert, which automatically sorts  the table. If a table has a large unsorted Region, a deep copy is much faster than a vacuum.

TRUNCATE commits immediately, even if it is inside a  transaction block. 

In most cases, you don't need to explicitly run the ANALYZE command. Amazon Redshift monitors  changes to your workload and automatically updates statistics in the background.

Automatic analyze is enabled by default.

Amazon Redshift can automatically sort and perform a VACUUM DELETE operation on tables in the  background.

Amazon Redshift automatically sorts data in the background to maintain table data in the order of  its sort key.

