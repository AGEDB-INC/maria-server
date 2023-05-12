## Identifier Names: Understanding Rules and Conventions

In database systems, identifiers such as databases, tables, columns, indexes, aliases, and views are used to define objects that make up a database schema. Identifiers have specific rules and conventions for naming to ensure they are unique and unambiguous. In this article, we will discuss the rules and conventions for naming identifiers in MariaDB, a popular open-source relational database management system.
**Unquoted Identifiers**

Unquoted identifiers are alphanumeric sequences that consist of ASCII characters, extended Unicode characters, dollar sign, or underscore. These identifiers cannot begin with a numeral unless quoted, but they can contain numerals as long as they are not the first character. They are not case-sensitive unless quoted. For example, user_name and user_name are equivalent, but User_Name is different from the other two.
**Quoted Identifiers**

Quoted identifiers allow for more flexibility in naming objects in a database schema. They can include all printable characters except the null character (U+0000) and the ASCII double quote character ("). Quoted identifiers can also contain the quote character if it is quoted. When using quoted identifiers, they are case-sensitive, meaning that user_name and User_Name are two different identifiers.

Quote Characters MariaDB supports using two types of quote characters for quoted identifiers. The backtick character (`) is the default, and it can be used to enclose any identifier, even if it contains spaces or special characters. Alternatively, the ANSI_QUOTES SQL_MODE flag can be set to allow the use of double quotes (") for quoting identifiers. If the MSSQL flag is set, square brackets ([ and ]) can be used for quoting.

Further Rules The following rules also apply to identifier names in MariaDB:
1. Identifier names are stored in Unicode (UTF-8).
2. Database, table, and column names cannot end with space characters.
3. Identifier names cannot contain the ASCII null character (U+0000) and supplementary characters (U+10000 and higher).
4. User variables cannot be used as an identifier or as part of an identifier in SQL statements.
5. Names such as 5e6 and 9e are not prohibited, but it is strongly recommended not to use them, as they could lead to ambiguity in certain contexts.

Maximum Length Identifiers in MariaDB have a maximum length of 64 characters for databases, tables, columns, indexes, constraints, stored routines, triggers, events, views, tablespaces, servers, and log file groups. Compound statement labels have a maximum length of 16 characters, while aliases have a maximum length of **256 characters**, except for column aliases in CREATE VIEW statements, which are checked against the maximum column length of **64 characters**(not the maximum alias length of 256 characters). Usernames have a maximum length of 80 characters, and roles have a maximum length of 128 characters. Multi-byte characters do not count extra towards the character limit.
Multiple Identifiers MariaDB allows the use of multiple identifiers to reference a specific object in a database schema. A period (.) is used to separate identifiers, and the period can be surrounded by spaces. For example, test.t1.i references column i in table t1 within the test database. This allows for the creation of fully qualified names for objects in a database schema, eliminating ambiguity when referencing them.
So Naming conventions and rules for identifiers in MariaDB ensure that object names are unique, unambiguous, and easy to use in SQL statements. Understanding these conventions and following best practices for naming database objects can make it easier to manage a database schema and improve the overall organization and clarity of the code.

## Identifier Case-sensitivity
When working with MariaDB, it's important to understand the rules around identifier case-sensitivity. This is partly determined by the underlying operating system, with Unix-based systems being case-sensitive and Windows being case-insensitive by default (though Mac OS X can be configured either way).
In MariaDB, database and table names, aliases, and trigger names are affected by the system's case-sensitivity, while index, column, column aliases, stored routine, and event names are never case-sensitive. The lower_case_table_names server system variable plays a key role in determining whether table and database names are compared in a case-sensitive manner.
By default, on Unix-based systems, this variable is set to 0, meaning that table and database names are compared in a case-sensitive manner. On Windows, it is set to 1, meaning that names are stored in lowercase and not compared in a case-sensitive manner. On Mac OS X, it is set to 2, meaning that names are stored as declared but compared in lowercase.
While it is possible to make Unix-based systems behave like Windows and ignore case-sensitivity, the reverse is not true, as the underlying Windows filesystem cannot support this. Even on case-insensitive systems, it is important to use the same case consistently within the same statement.
It's important to note that lower_case_table_names is a database initialization parameter, meaning that it must be set before running mysql_install_db and will not change the behavior of servers unless applied before the creation of core system databases.
Overall, understanding the rules around identifier case-sensitivity in MariaDB is an important part of developing and maintaining databases in this system.

## Binary Literals
Binary literals in MySQL can be represented in three formats - b'value', B'value' or 0bvalue. These literals are used to represent binary strings composed of 0 and 1 digits. They are useful when working with VARBINARY, BINARY, or BIT values.
To print the binary string value of a binary literal, use the SELECT statement followed by the binary literal. 

For example, 
	                SELECT 0b1000001 will print the binary string 'A'.

To convert a binary literal to an integer value, simply add 0 to it. 
For instance, 
	                SELECT 0b1000001+0 will return the integer value 65.
                    
Binary literals are useful in situations where binary strings need to be represented explicitly in a query. They are particularly useful in conjunction with the BIT data type.

## Boolean Literals
* In MariaDB, FALSE is equivalent to 0 and TRUE is equivalent to 1.
* These constants are not case-sensitive, so TRUE, True, and true all mean the same thing.
* When used with the IS operator, TRUE and FALSE are not synonyms of 1 and 0. For example, the expression 10 IS TRUE would return 1, but the expression 10 = * TRUE would return 0 because 1 is not equal to 10.
* The IS operator also accepts a third constant: UNKNOWN, which is always a synonym for NULL.
* While TRUE and FALSE are reserved words in MariaDB, UNKNOWN is not.
In general, it's important to be aware of these when working with Boolean expressions in MariaDB to avoid unexpected results.
## Date and Time Literals
#### How to use Date and Time literals in MariaDB
MariaDB supports the standard SQL syntax and ODBC syntax for defining Date, Time, and DateTime literals. These literals are useful for inserting and updating date and time values in tables, and for comparing dates and times in queries.
#### Date Literals
A Date literal is a string that represents a date value in the format 'YYYY-MM-DD' or 'YY-MM-DD'. You can use any punctuation character as a delimiter, but all delimiters must consist of one character. Different delimiters can be used in the same string, and delimiters are optional. A date literal can also be an integer in the format 
* YYYYMMDD or 
* YYMMDD.
For example, the following date literals are all valid and represent the same value:
* '1994-01-01' 
* '94/01/01' 
* '1994-01/01' 
* 19940101
#### DateTime Literals
A DateTime literal is a string that represents a date and time value in the format 'YYYY-MM-DD HH:MM:SS' or 'YY-MM-DD HH:MM:SS'. You can use any punctuation character as a delimiter for the date part and time part, but all delimiters must consist of one character. Different delimiters can be used in the same string, and delimiters are mandatory. The delimiter between the date part and time part can be a T or any sequence of space characters.
A DateTime literal can also be an integer in the format 
* YYYYMMDDHHMMSS
* YYMMDDHHMMSS
* YYYYMMDD 
* YYMMDD. 
In this case, all the time subparts must consist of two digits.
For example, the following DateTime literals are all valid and represent the same value:
* '1994-01-01T12:30:03' 
* '1994/01/01\n\t 12+30+03'
* '1994/01\01\n\t 12+30-03'
#### Time Literals
A Time literal is a string that represents a time value in the format 'D HH:MM:SS', 'HH:MM:SS, 'D HH:MM', 'HH:MM', 'D HH', or 'SS'. D is a value from 0 to 34 which represents days. The only allowed delimiter for Time literals is ':', and delimiters are mandatory except in the 'HHMMSS' format.
A Time literal can also be an integer in the format HHMMSS, MMSS, or SS.
For example, the following Time literals are all valid and represent the same value:
* '09:05:00' 
* '9:05:0' 
* '9:5:0' 
* '090500'
#### Special Values
MariaDB allows some special values for date and time literals, such as '0000-00-00' for Date, '00:00:00' for Time, and '0000-00-00 00:00:00' for DateTime. These values are only allowed if the SQL_MODE NO_ZERO_DATE flag is not set.
If the ALLOW_INVALID_DATES flag is set, invalid dates such as '30th February' are allowed. Otherwise, if the NO_ZERO_DATE flag is set, an error is produced, and if not, a zero-date is returned.
By understanding the syntax and format of Date and Time literals, you can efficiently work with date and time values in MariaDB.

## Hexadecimal Literals

Hexadecimal literals are a way to represent values using base-16 numbers, where each digit can range from 0 to 9 or from A to F. This notation can be used to represent characters as binary strings or to represent integers in a numeric context.

In MariaDB, there are three ways to write hexadecimal literals:
* x'value'
* X'value' and
* 0xvalue. 

The first two are SQL standard syntaxes, while the last one is a MySQL/MariaDB extension. The first two syntaxes always behave as a string, while the last one behaves as a string or as a number depending on context. It's important to note that hexadecimal literals can't be decimal numbers.

When used in a numeric context, hexadecimal literals are interpreted as integers. For example, the expression "0xF" represents the integer value 15. In a string context, they are interpreted as binary strings, where each pair of digits represents a character. For example, the expression "x'61'" represents the character 'a'.

In some cases, there can be differences between MariaDB and MySQL when dealing with hexadecimal literals. For example, the expression "x'0a'+0" returns 0 in MariaDB and 10 in MySQL. This is because MariaDB treats hexadecimal literals in a string context, while MySQL treats them as numbers.
Overall, hexadecimal literals are a useful notation for representing values in a compact and readable way, but it's important to be aware of their behavior in different contexts and between different database systems.

## Identifier Qualifiers

In SQL, identifiers are used to reference data structures such as databases, tables, or columns. Qualifiers are used to specify the context within which the final identifier is interpreted. Let's take a closer look at how identifier qualifiers work in SQL.
* Qualifiers for Databases When referencing a database, only the database identifier needs to be specified. For example, "db_name" is a valid qualifier for a database. If no database is specified, the current database is assumed. However, if there is no default database and no database is specified, an error is issued.

* Qualifiers for Objects within Databases For objects such as tables, views, functions, etc. that are contained within a database, the database identifier can be specified. If no database is specified, the current database is assumed. It's important to note that the database identifier is optional for objects within the current database.

* Qualifiers for Column Names For column names, the table and the database are generally obvious from the context of the statement. However, it is possible to specify the table identifier or the database identifier plus the table identifier. For example, "db_name.tbl_name.col_name" is a fully-qualified identifier that specifies the database, table, and column names.

* Quoting and Spacing All identifiers can be quoted individually using backticks (`) or other quote characters, if needed. Extra spacing, including new lines and tabs, is allowed between identifiers and qualifiers for readability.

* Dot (.) as Separator If a qualifier is composed of more than one identifier, a dot (.) must be used as a separator. For example, "db_name.tbl_name" uses a dot as a separator between the database and table identifiers. It's important to use the dot as a separator for correct interpretation of qualifiers.

* Dot (.) for Default Database In some cases, a table identifier may be prefixed with a dot (.) for ODBC compliance, but this has no practical effect on MariaDB. It is equivalent to just specifying the table name without the dot. For example, "tbl_name" is equivalent to ".tbl_name" or ".tbl_name".

Understanding how identifier qualifiers work in SQL is crucial for correctly referencing databases, tables, and columns in SQL statements. By using the appropriate qualifiers and following the correct syntax, you can ensure accurate interpretation of identifiers in your SQL queries.

## Identifier to File Name Mapping

When mapping SQL identifiers to file names on a filesystem, certain characters may not be allowed in file names, depending on the specific filesystem used. To address this issue, MariaDB uses a special character set to encode "potentially unsafe" characters in the identifier to derive the corresponding file name.
The encoding process involves converting the identifier to the "filename" character set, which then produces the file name. Conversely, converting the file name from the "filename" character set back to the original character set produces the identifier.
If the identifier only contains basic Latin letters, numbers, and underscores, the encoding matches the name. Otherwise, the identifier is encoded based on a table that maps Unicode character ranges to specific patterns. 

For example, the Latin-1 Supplement and Latin Extended-A character ranges are encoded as [@][0..4][g..z], while the Greek and Coptic character range is encoded as [@][5..9][g..z].
This encoding process happens transparently at the filesystem level, except for identifiers created before MySQL 5.1.6. For such identifiers, which have not been updated to the new encoding scheme, the server prefixes mysql50 to the identifier name.
To find the file name for a table with a non-Latin1 name, you can use the following query:

```
SELECT CAST(CONVERT("this_is_таблица" USING filename) AS binary);
```

This will return the file name, which is "this_is_@y0@g0@h0@r0@o0@i1@g0".

To find the table name for a given file name, you can use the following query:

```
SELECT CONVERT(_filename "this_is_@y0@g0@h0@r0@o0@i1@g0" USING utf8);
```

This will return the original table name, which is "this_is_таблица".
For old identifiers created before MySQL 5.1.6, you need to supply the mysql50 prefix to reference the table. For example:
SHOW COLUMNS FROM `#mysql50#table@1`;
This will display the columns of the table named "table@1" that was created before MySQL 5.1.6.


## MariaDB Error Codes

MariaDB shares many error codes with MySQL, it also has its own set of specific error codes. Understanding error codes is important for troubleshooting database issues, as it provides insight into what went wrong and how to fix it.
#### Shared MariaDB/MySQL Error Codes 
MariaDB and MySQL share error codes from 1000 to 1800. These codes include common errors such as syntax errors, connection errors, and permission errors. Here are some of the most common shared error codes:
* Error Code 1045: Access denied for user 'user'@'host' (using password: YES) This error is caused when the user does not have sufficient permissions to access a specific database or table. It may also indicate an incorrect username or password.

* Error Code 1064: You have an error in your SQL syntax This error is caused by a syntax error in an SQL statement. It could be a missing or extra character, a typo, or incorrect syntax.

* Error Code 1146: Table 'database.table' doesn't exist This error is caused when a query is trying to access a non-existent table. It could be a typo in the table name or a missing CREATE TABLE statement.

* Error Code 1364: Field 'field_name' doesn't have a default value This error is caused when a query is trying to insert a row into a table without specifying a value for a non-null field that does not have a default value.


#### MariaDB-Specific Error Codes
In addition to the shared error codes, MariaDB has its own set of specific error codes. These codes are numbered from 1900 and up. Some of the most common MariaDB-specific error codes include:

* Error Code 1905: Lock wait timeout exceeded; try restarting transaction This error is caused when a transaction is waiting for a lock to be released, but the lock is held by another transaction for too long. It can be fixed by either increasing the lock wait timeout or by optimizing the queries to reduce the time needed to acquire the lock.

* Error Code 1932: The used command is not allowed with this MariaDB version This error is caused when a command that is not supported by the current version of MariaDB is used. It could be an outdated query or a new feature that is not yet supported.

* Error Code 2002: Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2) This error is caused when the MySQL server is not running or when the connection to the server is refused. It could also indicate an incorrect socket file path.

* Error Code 2013: Lost connection to MySQL server during query This error is caused when the connection to the server is lost while a query is being executed. It could be caused by network issues, server overload, or a timeout.

For detailed error codes refer : https://mariadb.com/kb/en/mariadb-error-codes/#shared-mariadbmysql-error-codes
