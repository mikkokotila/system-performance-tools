# system-performance-tools
List of tools for investigating system performance

### Basic System Benchmarking

#### get a topline view to the system

`top`, `htop` or `atop` ... each of which are slightly different.

#### perform a comprehensive system benchmarking test

`perf bench`

NOTE: there are many useful options in the `perf` package, so do make yourself familiar with it. You can, for example, see exactly how a given command is using system resources.

### get a view of the memory

See available memory with: 

`watch free -m`

Or get a deep dive with:

`vmstat`

#### get a view of system utilization rate

`watch iostat`

NOTE: This is particularly useful when you are running something simultanously.

### Investigating Failing Requests

#### see the TOP10 services based on number of open sockets 

`socklist | tr -s ' ' | cut -d ' ' -f2,7 | sort | uniq -c | sort -nr | head -10`

#### get a deeper view into the connections to the server

`netstat`

Here we can see if there are a lot of connections in `TIME_WAIT` status, and identify which ports are the ones waiting.

For example, if you identify that port 8585 has a lot of wait statuses, then you can dig deeper into that.

#### get the number of waiting connections for a given port

`sudo netstat -peanut | grep 8585 | wc -l`

#### get the process id and name for a given port

`sudo netstat -peanut | grep 8585`

#### the the process id for a

`sudo ps aux | grep 5103`

NOTE: the process id and name will be on the last row of the output after a list of all the open connections.

#### find out exactly what a given process name is running

`ps aux | grep process_name`

Here we will find out exactly what are the files in our system that are associated with the waiting connections.

Next, it's time to look at the associated programs (or parts of programs) carefully, and think why they might be causing the issues.


### Investigating Database Issues

#### get the current sizes of mysql databases

`du -h /var/lib/mysql`

#### get the data types for mysql table

```
USE database_name;
SHOW tables;
SHOW FIELDS FROM some_table_name;
```

What you are looking for here is generally speaking: 

**PROBLEM:** the main table is storing strings as `VARCHAR` or `CHAR`
**SOLUTION:** store strings in separate lookup table and store binary index in main table instead

**PROBLEM:** integer values are stored as `INT` or similar
**SOLUTION:** store integer values as `BIT` instead

**PROBLEM:** `VARCHAR` is used so database size can't be predicted 
**SOLUTION:** store all string values outside of the main table

**PROBLEM:** floating point values are stored as `FLOAT`
**SOLUTION:** store as `BIT` instead by using "minor currency" approach

There are many other things, but these are the most common issues. In summary: 

- never store string values in the main tables
- all numeric data is stored as `BIT`
- for any values that repeat often (e.g. website url) keep it in a separate table
- fix column sizes in the design phase, so database size (and performance) can be predicted

Before making any changes, it's a great idea to run a proper benchmarking test. 

#### run a baseline benchmarking test on mysql performance

First install MySQL Tuner:

```
wget http://mysqltuner.pl/ -O mysqltuner.pl
wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/basic_passwords.txt -O basic_passwords.txt
wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/vulnerabilities.csv -O vulnerabilities.csv
```

Then run it:

`perl mysqltuner.pl --host 127.0.0.1`

Make sure to take note of the various statistics regarding your current performance, and then compare it after making changes. 
