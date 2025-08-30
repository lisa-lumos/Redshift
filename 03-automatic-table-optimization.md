# 3. Automatic table optimization
Automatic table optimization automatically optimizes the design of tables by applying sort/distribution keys, without the need for manual intervention. 

Existing tables with a distribution style or sort key of AUTO are already enabled for automation. 

## Enabling, disabling, and monitoring automatic table optimization
By default, tables created without explicitly defining sort/distributions keys are set to AUTO. 

The system view SVV_ALTER_TABLE_RECOMMENDATIONS records the current Amazon Redshift Advisor recommendations for tables.

ENCODE AUTO is the default for tables.

When you use CREATE TABLE, ENCODE AUTO is disabled when you specify compression encoding for any column in the table.

We recommend that you create your tables with DISTSTYLE AUTO, SORTKEY AUTO.

## Column compression
skip

## Data distribution
skip
## Sort keys
## Table constraints
skip

