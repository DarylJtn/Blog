---
title: Performance Reporting 1
date: "2020-05-04"
description: "Data Wrangling"
---
The first step to dealing with performance issues in a codebase is visualising the issue and targeting problematic sections of code. 

Often times these reports are needed with quick turnarounds after a system failure and require to be updated daily. Using R, bash and python we can create a minimum viable product and build on top of it creating reports that change with the needs of stakeholders.

In the following examples we will imagine a system running a batch of programs some in series and some concurrently. The processes will run between 10pm to 4am and the objective of the reports will be to find parts of the system we can focus on to reduce the time the over all batch

## DATA WRANGLING

In this example the information on when a program starts and finishes is stored in a log file. A timestamp, status and program name is printed to the log. The log file is used much more than printing start and end times. We will need to use regular expressions to filter the data down into something more manageable then go about restructuring the data to meet our need

Here is an sample of how the log file looks

```
20:00:00 01 MAY 2020,START,BATCH
Starting batch
getting account
20:00:01 01 MAY 2020,START,PAYMENT
|sql select| SELECT * FROM main where date > '2020-05-01'
1000 processed
2000 processed
...
21:03:25 01 MAY 2020,START,LETTER-PROCESS
sent letters
21:05:25 01 MAY 2020,END,LETTER-PROCESS
finished letters 
uploading files
21:14:32 01 MAY 2020,END,PAYMENT
downloading bureau files
```

We want to end up with a csv table looking something like this:

| Program name  | Start Time    | End Time  |
| ------------- |:-------------:| ---------:|
| Payment       | 20:00:00      | 21:14:25  |
| LETTER-PROCESS| 20:00:01      | 21:05:25  |

The grep command can be used to extract the timestamp rows
This regular expression can get us the rows containing start and end times

```
[0-9]{2}:[0-9]{2}:[0-9]{2} [0-9]{2} [A-Z]{3} [0-9]{4},(START|END),

```
we can then use this command to extract a sorted list of the program name start and end time

_grep_ filters out the timestamp rows from the logs
_sort_ gets the file sorted by program name then by the start/end column
_awk_ prints out the data in the format we want

```bash
egrep '[0-9]{2}:[0-9]{2}:[0-9]{2} [0-9]{2} [A-Z]{3} [0-9]{4},(START|END),' log.txt | 
sort -t , -k 3,3 |  awk -F',' '$2 ~ "END" {OFS=","; print $3,prevTime,$1}{prevTime=$1}' > data.txt

```
 We now have the data in the correct format and we can use R and python to create our reports 

