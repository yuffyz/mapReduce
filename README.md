# MapReduce on Google Cloud 

This repo contains two Hadoop MapReduce programs that run on a 3-node clusters on Google Cloud. The programs are used to process a log file, which is read into lines of **IP Address, Time, Type of HTTP Request, Requested File, HTTP Version and Status, etc**.  

<img src="pic/data.png" width="500">

## Part I: Top-3 IP address

 Goal: output the top-3 IP addresses with the granularity of an hour. 
 
 ### Design

This program runs two rounds of mapreduce. 

1. In the 1st round, the program is built on top of **logstat2**. It takes the access.log file, then output the count of each IP addresses within each hour. The result is saved in `/logstat2/output/`. 

2. In the 2nd round, only the mapper is used. The result of the 1st round mapreduce will be the input of the mapper with following command: 

```bash
/usr/local/hadoop/bin/hdfs dfs -cp /logstat2/output/part-00000 /logstat3/input/
```

Next, specify only the mapper and no reducer is used with:

```bash
-D mapred.reduce.tasks=0
```

For the mapper program, the input lines are grouped by hours. Then it outputs the top-3 IP addresses for each hour (within the 24 hour range). 

The mapper first takes in the input line and seperate each line at the horizontal tab character `\t` and the whitespace `' '` into three strings: `hr, ip, count`. 

Then, it converts both `hr` and `count` into numeric values. 

Next, using python container `defaultdict`, the program maps the value `[ip, count]` into key `hr`. 

Finally, the program iteratively sort the count within 24 hours, and extracted the top 3-IP addresses with their counts. 

<img src="pic/part1.png" width="500">


## Part II: A database search

Goal: Your program should be able to accept parameters from users, such as 0-1, which means from time 00:00 to 01:00, and output the top-3 IP addresses in the given time period.

 ### Design

This program also runs two rounds of mapreduce. It takes two paramters between 0 to 24, and output the top-3 IP addresses in this time period. 

 In the 1st round, the program is built on top of *logstat2*. It takes the access.log file and output the count of each IP addresses within each hour. The result is saved in `/logstat2/output/`. 

In the 2nd round, only mapper is used. First, in the `test.sh` file, The result of first round mapreduce will be the input of the mapper: 

```bash
/usr/local/hadoop/bin/hdfs dfs -cp /logstat2/output/part-00000 /logstat3/input/
```

Then, specify only mapper and no reducer is used with: 

```bash
-D mapred.reduce.tasks=0
```

Next, the mapper takes two parameters of two integer value between 0-24:

```bash
-mapper '../../mapreduce-test-python/logstat4/mapper.py 3 6'
```
Here, the two parameters being passes are **3** and **6**, which means I'd like to output the top-3 IP addresses between 3 to 6 am. 
(p.s. the two paramters are read in as string, but converted into integer in the mapper)

For the mapper `mapper.py`, it read the input lines into a list of lists. For each list of lists, I used list comprehension to filter by the two hour parameters that were given. Then, the subset list is sorted and the top-3 IP addresses are extracted. 

<img src="pic/part2.png" width="500">



