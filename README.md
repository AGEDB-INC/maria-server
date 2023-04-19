# Build and Debug from source (Ubuntu 22.04)

## Install Dependencies
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
