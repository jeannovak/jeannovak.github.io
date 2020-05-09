# Error-based SQLi

How to exploit error-based SQLi?

Extracting information using error-based SQLi is really quick and its available in ORACLE, postgreSQL and MSSQL.

Most of the time the error message is not reflected in the webapp output, but it can be embedded in an email message or appended to a log file. We can use error-based SQLi to dump information about the database itself, like database names, schemas and useful data.

Schemas are databases which purpose is to describe all other user-defined databases.

