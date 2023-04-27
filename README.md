# Build and Debug from source using cmake (Ubuntu 22.04)

## Install Dependencies

Figuring configuration contents:
Firstly Create configuration file in this path '/etc/apt/sources.list.d/mariadb.list` using following command:

```
sudo gedit /etc/apt/sources.list.d/mariadb.list
```

then Save the following repository configuration contents in the file `/etc/apt/sources.list.d/mariadb.list`. Note that the configuration is valid for the mariadb branch 11.0 and Ubuntu 22.04:
```
# Retrieved from: https://mariadb.org/download/?t=repo-config

# MariaDB 11.0 [RC] repository list - created 2023-02-24 18:43 UTC
# https://mariadb.org/download/
deb https://mirror.its.dal.ca/mariadb/repo/11.0/ubuntu jammy main
deb-src https://mirror.its.dal.ca/mariadb/repo/11.0/ubuntu jammy main
deb https://mirror.its.dal.ca/mariadb/repo/11.0/ubuntu jammy main/debug
```

Install dependencies:
```
sudo apt-get install software-properties-common \
      devscripts \
      equivs \
      curl \
      git
sudo apt-get build-dep mariadb-server
```

Import the repository key and update apt. You should not get any error from the mariadb repository server:

```
sudo curl -o /etc/apt/trusted.gpg.d/mariadb_release_signing_key.asc 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo apt-get update

```

## References:
    1. https://mariadb.com/kb/en/generic-build-instructions/ 

## Build

Make sure you completed the previous steps without any failure before moving forward.

We will be working on 3 main directories `maria-server` (source directory), `maria-server-build` (build directory), and `maria-server-data` (data directory). You can choose any names. We will create them inside the home directory. You can choose any directory where root privilige is not required.
```
cd ~
```

Fork and, then clone the forked repo:

```
git clone https://github.com/{your-username}/maria-server
```

Create the build and data directory.
```
mkdir ~/maria-server-build
mkdir ~/maria-server-data
```

Run the following commands to start building. Notice the directory used in each command:
```
cd ~/maria-server-build
cmake ~/maria-server/ -DCMAKE_BUILD_TYPE=Debug
cmake --build ./ -j8
```

## Configure

Before proceeding, you should have mariadb installed without any error from cmake.

Copy the following mariadb configuration into the file `~/.my.cnf`. Set the absolute path to your data and build directory in the `datadir` and `language` entry before saving:
```
# Example MariadB config file.
# You can copy this to one of:
# /etc/my.cnf to set global options,
# /mysql-data-dir/my.cnf to get server specific options or
# ~/my.cnf for user specific options.
#
# One can use all long options that the program supports.
# Run the program with --help to get a list of available options

# This will be passed to all MariaDB clients
[client]
#password=my_password
#port=3306
#socket=/tmp/mysql.sock

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# The mariadb server  (both [mysqld] and [mariadb] works here)
[mariadb]
#port=3306
#socket=/tmp/mysql.sock

# The following three entries caused mysqld 10.0.1-MariaDB (and possibly other versions) to abort...
# skip-locking
# set-variable  = key_buffer=16M

loose-innodb_data_file_path = ibdata1:1000M
loose-mutex-deadlock-detector
gdb

######### Fix the two following paths

# Where you want to have your database
datadir={absolute-path-to-your-data-directory}

# Where you have your mysql/MariaDB source + sql/share/english
language={absolute-path-to-your-build-directory}/sql/share/english

########## One can also have a different path for different versions, to simplify development.

[mariadb-10.1]
lc-messages-dir=/my/maria-10.1/sql/share

[mariadb-10.2]
lc-messages-dir=/my/maria-10.2/sql/share

[mysqldump]
quick
set-variable = max_allowed_packet=16M

[mysql]
no-auto-rehash

[myisamchk]
set-variable= key_buffer=128M
```

Initialize data files in the data direcotry:
```
cd ~/maria-server-build
./scripts/mariadb-install-db --srcdir={absolute-path-to-your-source-directory} --user=$LOGNAME
```

## Run

Check whether the install was successful:
```
./sql/mariadbd
```

You should see something similar to this in the terminal:
```
...
...
2023-02-24 14:37:06 0 [Note] ./sql/mariadbd: ready for connections.
Version: '11.0.1-MariaDB-debug'  socket: '/tmp/mysql.sock'  port: 3306  Source distribution
```

Shutdown the server using `CTRL + C`.

While the server was running, you could also run the client using `./client/mariadb` to test SQL commands.

## Debug (GDB)

Run gdb and the server with debug flag:
```
gdb -tui --args ./sql/mariadbd --gdb
```

Inside gdb, set breakpoint at the `write_row` function. This function is likely used by the sql INSERT command:
```
b write_row
```

Start the server within gdb:
```
run
```

In a separate terminal, run the client:
```
./client/mariadb
```

Set up a sample table:
```
CREATE DATABASE mydb;
USE mydb;
CREATE TABLE mytable (id int);
```

Insert a row into mytable. If gdb is setup correctly, the INSERT command will pause the client and trigger the breakpoint in the `ha_innobase::write_row` function.
```
INSERT INTO mytable VALUES ('1');
```

## Debug (VSCode)

Use the following launch configuration. Make sure to set the build directory in the `program` entry. It should configure the default C\C++ extension to launch mariadb with debug flag:
```
"configurations": [
    ...,
    ...,
    {
        "name": "(gdb) Launch",
        "type": "cppdbg",
        "request": "launch",
        "program": "{absolute-path-to-your-build-directory}/sql/mariadbd",
        "args": ["--gdb", "--debug"],
        "stopAtEntry": false,
        "cwd": "${fileDirname}",
        "environment": [],
        "externalConsole": false,
        "MIMode": "gdb",
        "setupCommands": [
            {
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            },
            {
                "description": "Set Disassembly Flavor to Intel",
                "text": "-gdb-set disassembly-flavor intel",
                "ignoreFailures": true
            }
        ]
    }
]
```

Launch the server from the VSCode's Run and Debug tab. You can set breakpoints from the IDE's text editor. But, you have to run the sql commands from the client as shown in the previous section.

## Rebuild

If you want to rebuild, first go the the source directory and run this command:
```
cd ~/maria-server
git clean -xffd && git submodule foreach --recursive git clean -xffd
```

Then, follow all the previous steps from the beginning.


## Install MariaDB on Windows

There are few pre-requisite we need to install before even start building the server:

### **Visual C++**

Download and Install Visual C++ from [Microsoft](https://visualstudio.microsoft.com/). Community Version 2019 and 2022 are supported at this time.

Make sure to select and install "Desktop Development with C++" during the installation process.


![Visual Code Installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/574kwg6tz8xl994lqf96.png)


### **CMake**

Download CMake from [cmake-v3.26.0](https://github.com/Kitware/CMake/releases/download/v3.26.0/cmake-3.26.0-windows-x86_64.msi) and install on the machine. 

- Minimum version required: 3.14 

**Check out one of my blog on** [CMake installation](https://dev.to/arun3sh/install-cmake-on-windows-4eo5) 

Make sure to add `C:\GnuWin32\bin` to your windows system path. 

### **Git**

Download and Install git using [Git-2.39.2-64-bit.exe](https://github.com/git-for-windows/git/releases/download/v2.39.2.windows.1/Git-2.39.2-64-bit.exe)

**While installing git, make sure to select the PATH environment as follows:**
![Git Installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/08hgxcmd6pcmbqroo4hs.png)

### **Bison**

Download the executable from the link:
[Download Gnu Diff](https://ftp.gnu.org/gnu/diffutils/)

Follow the steps to install the executable.

## Build

- Get the source code

You will be able to find the source code of MariaDB from their GitHub repo [MariaDB Github Repository](https://github.com/MariaDB/server) 

- Clone the repo to your system

```bash
git clone git@github.com:MariaDB/server.git 
```

After the clone is complete, you will have a complete copy of the repository in the server directory. You can then work on the code locally, commit changes, and push them back to the remote repository as needed.

- Create Build directory

```bash
# create a directory with name "build-dir"
mkdir build-dir

# go into "build-dir"
cd build-dir

# Run the following commands in sequence

cmake ../server

# If you want to build with Debugging option
# otherwise just omit  --config Debug
cmake --build . --config Debug
```
Build will take a while to complete and make sure there are no errors while building.

```bash
# go to the directory
cd ..\build-dir\mysql-test

# Test your build
perl mysql-test-run.pl --suite=main --parallel=auto

```

If all the tests came out as passed, we have installed the MariaDB perfectly on Windows machine.

## Insert Command

The INSERT statement is used in MariaDB to add new data into a table. The basic syntax of the INSERT statement is as follows:
```
INSERT INTO table_name (column1, column2, column3, ...) VALUES (value1, value2, value3, ...);
```

For example, if you have a table called students with columns id, name, and age, and you want to insert a new student with an ID of 1, a name of "John", and an age of 20, you would use the following INSERT statement:

```
INSERT INTO students (id, name, age) VALUES (1, 'John', 20);
```

If you want to insert multiple rows at once, you can use a comma-separated list of values sets enclosed in parentheses. For example, to insert two students into the students table, you can use the following INSERT statement:

```
INSERT INTO students (id, name, age) VALUES (1, 'John', 20), (2, 'Jane', 22);
```

Using the SET clause:
```
INSERT INTO students SET id= 3, name = 'Doe',age=23;
```

SELECTing from another table:
```
INSERT INTO contractor SELECT * FROM person WHERE status = 'c';
```
This INSERT INTO statement with a SELECT subquery in MariaDB allows you to copy data from one table to another based on a specific condition.
In this case, the statement is selecting all columns (*) from the person table where the status column is equal to 'c', which stands for contractors. The SELECT statement acts as a filter to select only the rows that match the condition.
Then, the selected rows are inserted into the contractor table using the INSERT INTO statement. The columns in the person table and the contractor table must match in number and data type for this to work correctly.

## Buffer Pool

Buffer pool is an in-memory cache used for faster access to frequently accessed data. It stores data and indexes reducing disk I/O.

### Working of Buffer Pool

Buffer pool mechanism works on the the two sublist concepts the **new sublist** and the **old sublist** where every item when accessed first time gets on the top of old sublist and on being called while in the old sublist moves to the top of new sublist. By default **37%** of space is reserved for old-block. 

The mechanism of moving tuples out of the new sublist and old sublist is same as both of them works on the LRU algorithm. Once the block moves from old sublist to the head of new sublist then all the previously present blocks in new sublist move one step below and if the new sublist reaches its capacity then the least recently used block from new sublist moves to the head of old sublist.

### Size of Buffer Pool

The size of the buffer pool is determined by **innodb_buffer_pool_size** system variable. The size of buffer pool should be adjusted per your needs to see the best performance. In order to configure the size of buffer pool set **innodb_buffer_pool_size** system variable, the InnoDB Buffer Pool should usually be between 50%-75% of the memory available.

#### 2 ways to configure size of buffer pool

1) The size of the InnoDB buffer pool can be changed dynamically by setting the innodb_buffer_pool_size system variable using the SET GLOBAL statement which requires SUPER privilege.

2)Changing the size of buffer pool by setting **innodb_buffer_pool_size** system variable in configuration file. Ensure that your custom configuration file is read last by using the z- prefix in the file name so that changes made by you are not overwritten. 
innodb_buffer_pool_size system variable in the configuration file.
**innodb_buffer_pool_size** needs to be set in a group that will be read by MariaDB Server, such as [mariadb] or [server]. When set in a configuration file, the value supports units such as "M" (Megabyte), "G" (Gigabyte), etc. For this method server restart is needed to reflect the changes.

### Saving And Restoring Buffer Pool State

Innodb stores some percentage of most recently used pages from buffer pool at server shutdown and restores them at server restart. This is managed by **innodb_buffer_pool_dump_pct** configuration option which is used to reduce warmup period.
**innodb_buffer_pool_dump_at_shutdown** and **innodb_buffer_pool_load_at_startup** system variables should be enabled to allow buffer pool dump at shutdown and restore at server restart.

#### Some Important Points Related to Buffer Pool

* Pages are evicted using a least recently used (LRU) algorithm.

* InnoDB reserves additional memory for buffers and control structures, so that the total allocated space is approximately 10% greater than the specified buffer pool size.

* The size of each page in the Buffer Pool depends on the value of the **innodb_page_size** system variable.

# JSON(JavaScript Object Notation)

## JSON Introduction

JSON stands for JavaScript object notation. It is a text format for storing and transporting data. It is an easy concept to learn.

### Example

Following is an example of how we write JSOn string:

```
'{"product":"Clothes", "id":"331134", "color":"Blue"}'
```

It defines an object with 3 properties:
1. product
2. id
3. color

Each property above has a value.
You can access the data as an object, if you parse above string with a JavaScript program.

```
let product = obj.product
let color = obj.color
```

## What is JSON? Why should we use it?

JSON stands for JavaScript object notation. It is a lightweight data interchange format and is written in JavaScript object notation. It is also used for interchanging data between computers and is also language independent. JSON syntax is derived from JavaScript object notation but it is text only. COde for reading and generating Many programming languages provide utility of reading and generating JSON.

JSON format and creating of JavaScript objects are syntactically identical and due to this, JavaScript programs can convert JSON data into JS objects. Since the format of JSON is text only, JSON data can easily be sent between computers and used by any programming language. JavaScript has some built in functions that converts JSON strings into JavaScript objects.

```
JSON.parse() and JSON.stringify()
```

JSON.parse is used for conversion of JSON strings into JavaScript objects and JSON.stringify is used for converting an object into a JSON string. You can send and receive a JavaScript object from the server in pure text.

## JSON Syntax Rules

The JSON syntax is a subset of JS syntax.

### Syntax Rules

Syntax of JSON is derived from JS object notation syntax:
* Data is placed in key/value pairs
  
  The data is written in key value pairs and this pair consists of a name, followed by a colon and then followed by a value.
* Data is separated by commas
  
  Data i.e. key value pairs are separated by commas to separate them.
* Curly brackets to hold objects

  Curly braces i.e. {} are used for storing objects in JSON.
* a brackets to hold arrays

  We use square brackets [] to store arrays in JSON
  

## JSON format

JSON format requires key value pairs, i.e. the keys must be strings written inside double quotes. A key value pair consists of a name, followed by a colon, followed by a value.

```plaintext
## Example

"Color" : "Blue"
```

In above example Color is the key and blue is the value of that key.

## JSON Datatypes

JSON has following datatypes:

* String
* Number
* Object
* Array
* Boolean
* Null

JSON datatype cannot be any of the following:
* function
* date
* undefined

### String:
String values should be enclosed in double quotes
For example:

```plaintext
{"Name":"Talha"}
```

### Number:
Digits in JSON can be any integer or floating point numbers.
For example:

```plaintext
{"id":29871}
```

### Object:
Values used in JSON can be objects as well:
For example:

```plaintext
{
    "Employee":
        {"name":"Talha", "age":10, "id":1234}
}
```

### Arrays:
Values in JSON can be arrays as well.
For example:

```plaintext
{
"employees":["John", "Anna", "Peter"]
}
```

### Booleans:
Values in JSON can be true/false.
For example:

```plaintext
{"Married":true}
```

### Null:
Values in JSON can be Null as well.
For example:

```plaintext
{"Date of birth": null}
```

## RESEARCH for OQGraph

### What is OQGraph?
OQGraph is a storage engine for MariaDB that is designed specifically for storing and querying graphs. It allows you to store graph data directly in the database and perform complex graph queries using SQL syntax.

OQGraph is based on the property graph model, which means that each vertex and edge in the graph can have arbitrary properties attached to it. This makes it a flexible model that can be used to represent a wide variety of data. OQGraph is also highly optimized for both storage and query performance, using the Boost Graph Library (BGL) to achieve this.

### How does OQGraph work?
OQGraph stores graph data in a set of tables optimized for efficient storage and retrieval. Vertices are stored in a separate table, with their properties stored as columns in the table. Edges are stored in a separate table, with each row representing a single edge in the graph. Each edge has a source vertex and a destination vertex, represented as foreign keys to the vertex table.

OQGraph uses recursive common table expressions (CTEs) to perform graph traversal queries. A CTE is a temporary result set that is defined within the execution of a single SQL statement. The recursive CTE in OQGraph allows queries to traverse the graph by repeatedly joining the edge table to itself, building up a path from the starting vertex to the desired destination vertex.

### Using OQGraph
To use OQGraph, you first need to create tables to store your graph data. You can then specify the OQGRAPH engine for the table that will store the edges of the graph. Once your tables are set up, you can write SQL queries that use CTEs to traverse the graph and retrieve the desired data.

# Implementing cypher UDF function

Overall goal is to execute cypher query within SQL like this: `SELECT * FROM cypher(...);`.

We need to find a way to implement the `cypher` UDF which returns a set of rows.

## Function vs. Procedure vs. UDF

Quick summary of the ways of creating functions\procedures in MariaDB.

| Type             | Written in | Returns     |  Usage |Example |
|------------------|------------|-------------|--------|-|
| [Stored function](https://mariadb.com/kb/en/stored-functions/)   | SQL        | Single value                      | In any SQL statement      | `SELECT my_function();`|
| [Stored procedure](https://mariadb.com/kb/en/stored-procedures/) | SQL        | Nothing                           | Only in CALL statement    | `CALL my_function();`  |
| [UDF](https://mariadb.com/kb/en/user-defined-functions/)         | C\C++      | STRING, INTEGER, REAL or DECIMAL  | In any SQL statement      | `SELECT my_udf();`|

## Idea 1

MariaDB UDF functions currently support returning only 4 primitive types.

So, we can implement `cypher` as a stored procedure. The proeduce will execute following tasks:

1. It will call a helper UDF that returns JSON string containing number of columns and type of each column to be returned by the cypher query.
2. It will create a temporary table based on the previous information.
3. It will call the 'executor' UDF function and pass the temporary table. The UDF will put the result of cypher query into the temporary table.
4. It will execute `SELECT * FROM temporary_table;`.

## Idea 2

We can work on the following MariaDB issue which is about implementing support for functions that can return set of rows. Once the implementation is merged into MariaDB repo, we can implement cypher UDF just like we did in Postgresql.

<https://jira.mariadb.org/browse/MDEV-5199>.

