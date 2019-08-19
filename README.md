## system-performance-tools
List of tools for investigating system performance

## Investigating Failing Requests

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
