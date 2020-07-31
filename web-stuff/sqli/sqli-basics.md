# SQLi Basics

What you need to learn:

1. SQL statement syntax
2. How to perform a query
3. How to union the results of two queries
4. The DISTINCT and ALL operator
5. How comments work

SQL statement structure

```text
> SELECT <column list> FROM <table> WHERE <conditions>

> SELECT name, description FROM products WHERE id=9;
```

UNION performs an union between two results:

```text
> <select statement> UNION <second select statement>;
```

If a table contains duplicate rows, use the DISTINCT operator to filter out duplicate entries

```text
> SELECT DISTINCT <field list> <remainder of statement>;
```

UNION statement implies DISTINCT by default, which can be prevented by the ALL operator:

```text
> <SELECT statement> UNION ALL <other SELECT statement>;
```

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

