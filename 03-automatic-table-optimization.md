# 3. Automatic table optimization
Automatic table optimization is a self-tuning capability, that automatically optimizes the design of  tables by applying sort and distribution keys without the need for administrator intervention. 

By using automation to tune the design of tables, you can get started and get the fastest performance  without investing time to manually tune and implement table optimizations. 

Existing tables with a  distribution style or sort key of AUTO are already enabled for automation. 

## Enabling, disabling, and monitoring automatic table optimization
By default, tables created without explicitly defining sort keys or distributions keys are set to AUTO. 

The system view SVV_ALTER_TABLE_RECOMMENDATIONS records the current Amazon Redshift  Advisor recommendations for tables.

ENCODE AUTO is the default for tables.

When you use CREATE TABLE, ENCODE AUTO is disabled when you specify compression encoding  for any column in the table.

We recommend that you create your tables with DISTSTYLE AUTO. 

We recommend that you create your tables with SORTKEY AUTO.

## Column compression
skip

## Data distribution
skip
## Sort keys
## Table constraints
skip
































