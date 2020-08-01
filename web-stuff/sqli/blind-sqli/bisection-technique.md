# Bisection Technique

To speed up the SQLi, we can use a technique named Bisection, which reduces the number of queries by narrowing down the number of characters in the charset that you need to query. This means that you need to be able to understand the nature of the character you are trying to guess:

* \[A-Z\]
* \[a-z\]
* \[0-9\]

For that, we will be using the following functions:

ASCII\(\) returns the ASCII code of a character. UPPER\(\) function transforms a character into uppercase. LOWER\(\) function transforms a character into lowercase.

The first test is too see if the conversion to uppercase of the current character will yield a FALSE or TRUE condition:

```text
ASCII(UPPER(SUBSTRING((), , 1)))=
```

Then, we test if the conversion to lowercase of the current character will yield a FALSE or TRUE condition:

```text
ASCII(LOWER(SUBSTRING((), , 1)))=
```

Now we need to evaluate the results:

* If the first query returns TRUE and the second is FALSE, the character is uppercase. So we will iterate through \[A-Z\] 
* If the second query returns FALSE and the second is TRUE, the character is lowercase. So we will iterate through \[a-z\] 
* If both queries are TRUE, the character is either a number or a symbol. So we will iterate through \[0-9\] and symbols only.

Now, for example, you discovered that the character is lowercase. After that, you need to detect the first character. One way to decrease your scope by asking the server if the character ASCII value is higher or lower than an arbitrary ASCII value:

```text
' OR ASCII(SUBSTRING(user(),1,1)) <=109
```

For example, right now, we're asking if the ASCII value of the first character of the username is lower than 109. Lowercase ASCII values goes from 97-122. In this case, 109 would be M, which is in the middle of the alphabet.

* If the application returns FALSE, you now the first character ASCII value is higher than 109, that means it can only be N-Z. 
* If it returns TRUE, you now the first character ASCII value is lower than 109, that means it can only be A-M.

This can be repeated indefinitely until you discover the first char. Always halving the scope of ASCII characters.

