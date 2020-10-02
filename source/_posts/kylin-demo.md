---
title: Kylin Demo
date: 2020-07-09 16:37:28
categories: [Big Data]
tags: [Docker, Hive, Kylin]
---
First and formost, Login in server 115.

# Docker image and container

## Image
``` bash
[root@c8-2 aemg]# docker images
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
kylin_sqoop                                     20200708            9dffd69df6f6        24 hours ago        3.91GB
apachekylin/apache-kylin-standalone             3.1.0               2ce49ae43b7e        6 days ago          2.56GB
docker.elastic.co/elasticsearch/elasticsearch   7.8.0               121454ddad72        3 weeks ago         810MB
```
> #### kylin_sqoop is the image integrated with kylin and sqoop tool.

<!--more-->

## Container
``` bash
[root@c8-2 aemg]# docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                NAMES
6f3eb07ad486        kylin_sqoop:20200708                                  "/home/admin/entrypo…"   24 hours ago        Up 24 hours         0.0.0.0:7070->7070/tcp, 0.0.0.0:8032->8032/tcp, 0.0.0.0:8042->8042/tcp, 0.0.0.0:8088->8088/tcp, 0.0.0.0:16010->16010/tcp, 0.0.0.0:50070->50070/tcp   kylin_sqoop
388e9b6f5a75        docker.elastic.co/elasticsearch/elasticsearch:7.8.0   "/tini -- /usr/local…"   6 days ago          Up 5 days           0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                                                                                                       elasticsearch
```
> #### "6f3eb07ad486" is the kylin_sqoop image's container, and the container also named as kylin_sqoop.

## Enter the container with bash
``` bash
[root@c8-2 aemg]# docker exec -it kylin_sqoop bash
[root@6f3eb07ad486 admin]# pwd
/home/admin
```
# Hive

Submit the following SQL to hive, check the result and execution time.

``` sql
select provinces.NAME, sum(score_level.number_of_student) from score_level left join provinces on provinces.id = score_level.province where score_level.art_science_division = 'science' and score_level > 600 group by provinces.NAME;
```
``` bash
[root@6f3eb07ad486 admin]# hive
ls: cannot access /home/admin/spark-2.3.1-bin-hadoop2.6/lib/spark-assembly-*.jar: No such file or directory

Logging initialized using configuration in jar:file:/home/admin/apache-hive-1.2.1-bin/lib/hive-common-1.2.1.jar!/hive-log4j.properties
hive> show databases;
OK
default
testdb
Time taken: 0.555 seconds, Fetched: 2 row(s)
hive> use testdb;
OK
Time taken: 0.011 seconds
hive> show tables;
OK
admission_level
ncee_fraction_lines
provinces
score_level
Time taken: 0.022 seconds, Fetched: 4 row(s)
hive> select provinces.NAME, sum(score_level.number_of_student) from score_level left join provinces on provinces.id = score_level.province where score_level.art_science_division = 'science' and score_level > 600 group by provinces.NAME;
Query ID = root_20200709015328_9a2d2050-14f5-4ef7-a92a-245d71ecc103
Total jobs = 1
20/07/09 01:53:30 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Execution log at: /tmp/root/root_20200709015328_9a2d2050-14f5-4ef7-a92a-245d71ecc103.log
2020-07-09 01:53:31	Starting to launch local task to process map join;	maximum memory = 477626368
2020-07-09 01:53:31	Dump the side-table for tag: 1 with group count: 34 into file: file:/tmp/root/4db548ac-9d3a-415a-ad95-4727dc362bcf/hive_2020-07-09_01-53-28_267_846283123739194736-1/-local-10004/HashTable-Stage-2/MapJoin-mapfile01--.hashtable
2020-07-09 01:53:31	Uploaded 1 File to: file:/tmp/root/4db548ac-9d3a-415a-ad95-4727dc362bcf/hive_2020-07-09_01-53-28_267_846283123739194736-1/-local-10004/HashTable-Stage-2/MapJoin-mapfile01--.hashtable (1177 bytes)
2020-07-09 01:53:31	End of local task; Time Taken: 0.473 sec.
Execution completed successfully
MapredLocal task succeeded
Launching Job 1 out of 1
Number of reduce tasks not specified. Estimated from input data size: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1594171348472_0010, Tracking URL = http://6f3eb07ad486:8088/proxy/application_1594171348472_0010/
Kill Command = /home/admin/hadoop-2.7.0/bin/hadoop job  -kill job_1594171348472_0010
Hadoop job information for Stage-2: number of mappers: 1; number of reducers: 1
2020-07-09 01:53:36,761 Stage-2 map = 0%,  reduce = 0%
2020-07-09 01:53:40,852 Stage-2 map = 100%,  reduce = 0%, Cumulative CPU 1.77 sec
2020-07-09 01:53:43,919 Stage-2 map = 100%,  reduce = 100%, Cumulative CPU 2.81 sec
MapReduce Total cumulative CPU time: 2 seconds 810 msec
Ended Job = job_1594171348472_0010
MapReduce Jobs Launched: 
Stage-Stage-2: Map: 1  Reduce: 1   Cumulative CPU: 2.81 sec   HDFS Read: 3852245 HDFS Write: 371 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 810 msec
OK
Anhui	59179
Beijing	39746
Chongqing	52969
Fujian	12009
Gansu	2165
Guangdong	16384
Guangxi	20751
Guizhou	11409
Hainan	48315
Hebei	86680
Heilongjiang	36847
Henan	63351
Hubei	40833
Hunan	37358
Jiangxi	17869
Jilin	22894
Liaoning	39435
Neimenggu	14947
Ningxia	1653
Qinghai	903
Shaanxi	21635
Shandong	109746
Shanxi	15581
Sichuan	93480
Tianjin	15560
Yunnan	19241
Zhejiang	74781
Time taken: 17.742 seconds, Fetched: 27 row(s)
```
We can see, it costs 17.742 seconds to get the result.

# Comparison

http://{host}:7070/kylin/query

submit the same SQL to kylin, compare the execution time with hive.
<!-- ![kylin execution](http://doc.aemg-online.net/server/../Public/Uploads/2020-07-09/5f0688f2ea821.png "kylin execution") -->
![kylin execution](/blog/img/kylin-insight.png "kylin execution")

> Duration: 1.54s, which is faster than hive.

# Other links

Kylin Web UI: http://{host}:7070/kylin/login
Hdfs NameNode Web UI: http://{host}:50070
Yarn ResourceManager Web UI: http://{host}:8088
HBase Web UI: http://{host}:16010