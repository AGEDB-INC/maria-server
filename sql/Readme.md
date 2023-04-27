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