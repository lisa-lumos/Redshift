# 6. User defined functions
Because built-in functions exist in the system catalog schema,  pg_catalog, you can create a UDF with the same name in another schema, such as public or a user-defined schema.

UDF supports SQL and Python (can import custom modules). Also AWS Lambda UDFs. 

To create a UDF, you must have usage privilege on SQL or Python. By default, usage on SQL is granted to Public. Permission to run new UDFs is also granted to Public by default. 

A function is identified by its name and signature. 

Best practices:
- Name all UDFs using the prefix `f_`



















