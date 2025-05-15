# Redshift perspective guidance
## 0. Introduction
https://docs.aws.amazon.com/prescriptive-guidance/latest/query-best-practices-redshift/welcome.html

This guide is intended for data engineers, data architects, and data analysts who design or use tables and queries in Amazon Redshift.

## 1. Architecture components
skip 

## 2. Query performance factors
Sort keys: Amazon Redshift stores data on disk in sorted order according to a table's sort keys. The query optimizer and the query processor use the information about where the data is located within a compute node to reduce the number of blocks that must be scanned. This improves query speed significantly by reducing the amount of data to process. We recommend that you use sort keys to facilitate filters in the WHERE clause. 

Data compression: Reduces storage requirements, which reduces disk I/O and improves query performance. When you run a query, the compressed data is read into memory and then uncompressed when the query runs. By loading less data into memory, Amazon Redshift can allocate more memory to analyzing the data. Because columnar storage stores similar data sequentially, Amazon Redshift can apply adaptive compression encodings specifically tied to columnar data types. The best way to enable data compression on table columns is by using the AUTO option in Amazon Redshift to apply optimal compression encodings when you load the table with data. 

## 3. Best practices for tables

## 4. Best practices for queries

## 5. Best practices for Redshift Spectrum

## Resources

