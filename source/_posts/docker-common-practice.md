---
title: docker common practice
date: 2020-07-03 13:55:41
categories:
tags:
---
# 

<!--more-->

```
[root@c8-2 ~]# docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                NAMES
388e9b6f5a75        docker.elastic.co/elasticsearch/elasticsearch:7.8.0   "/tini -- /usr/local…"   2 hours ago         Up 2 hours          0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                                                                                                       elasticsearch
070dd7124d70        apachekylin/apache-kylin-standalone:3.0.1             "/home/admin/entrypo…"   8 days ago          Up 2 hours          0.0.0.0:7070->7070/tcp, 0.0.0.0:8032->8032/tcp, 0.0.0.0:8042->8042/tcp, 0.0.0.0:8088->8088/tcp, 0.0.0.0:16010->16010/tcp, 0.0.0.0:50070->50070/tcp   kylin

[root@c8-2 ~]# docker inspect -f "{{ .Config.Env }}" 070dd7124d70
[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/admin/jdk1.8.0_141/bin:/home/admin/hadoop-2.7.0/bin:/home/admin/apache-hive-1.2.1-bin/bin:/home/admin/hbase-1.1.2/bin:/home/admin/apache-maven-3.6.1/bin:spark-2.3.1-bin-hadoop2.6/bin:/home/admin/kafka_2.11-1.1.1/bin HIVE_VERSION=1.2.1 HADOOP_VERSION=2.7.0 HBASE_VERSION=1.1.2 SPARK_VERSION=2.3.1 ZK_VERSION=3.4.6 KAFKA_VERSION=1.1.1 LIVY_VERSION=0.6.0 JAVA_HOME=/home/admin/jdk1.8.0_141 MVN_HOME=/home/admin/apache-maven-3.6.1 HADOOP_HOME=/home/admin/hadoop-2.7.0 HIVE_HOME=/home/admin/apache-hive-1.2.1-bin HADOOP_CONF=/home/admin/hadoop-2.7.0/etc/hadoop HADOOP_CONF_DIR=/home/admin/hadoop-2.7.0/etc/hadoop HBASE_HOME=/home/admin/hbase-1.1.2 SPARK_HOME=/home/admin/spark-2.3.1-bin-hadoop2.6 SPARK_CONF_DIR=/home/admin/spark-2.3.1-bin-hadoop2.6/conf ZK_HOME=/home/admin/zookeeper-3.4.6 KAFKA_HOME=/home/admin/kafka_2.11-1.1.1 LIVY_HOME=/home/admin/apache-livy-0.6.0-incubating-bin KYLIN_VERSION=3.0.1 KYLIN_HOME=/home/admin/apache-kylin-3.0.1-bin-hbase1x]

[root@c8-2 ~]# docker commit --change "ENV SQOOP_HOME=/home/admin/sqoop-1.4.7.bin__hadoop-2.6.0" 070dd7124d70  apachekylin/apache-kylin-standalone:3.0.1
sha256:22b22aeded23dbf9548534cadadd5459360214569ba78b4987f4acc674367b32

[root@c8-2 ~]# docker inspect -f "{{ .Config.Env }}" 22b22aeded23dbf9548534cadadd5459360214569ba78b4987f4acc674367b32
[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/admin/jdk1.8.0_141/bin:/home/admin/hadoop-2.7.0/bin:/home/admin/apache-hive-1.2.1-bin/bin:/home/admin/hbase-1.1.2/bin:/home/admin/apache-maven-3.6.1/bin:spark-2.3.1-bin-hadoop2.6/bin:/home/admin/kafka_2.11-1.1.1/bin HIVE_VERSION=1.2.1 HADOOP_VERSION=2.7.0 HBASE_VERSION=1.1.2 SPARK_VERSION=2.3.1 ZK_VERSION=3.4.6 KAFKA_VERSION=1.1.1 LIVY_VERSION=0.6.0 JAVA_HOME=/home/admin/jdk1.8.0_141 MVN_HOME=/home/admin/apache-maven-3.6.1 HADOOP_HOME=/home/admin/hadoop-2.7.0 HIVE_HOME=/home/admin/apache-hive-1.2.1-bin HADOOP_CONF=/home/admin/hadoop-2.7.0/etc/hadoop HADOOP_CONF_DIR=/home/admin/hadoop-2.7.0/etc/hadoop HBASE_HOME=/home/admin/hbase-1.1.2 SPARK_HOME=/home/admin/spark-2.3.1-bin-hadoop2.6 SPARK_CONF_DIR=/home/admin/spark-2.3.1-bin-hadoop2.6/conf ZK_HOME=/home/admin/zookeeper-3.4.6 KAFKA_HOME=/home/admin/kafka_2.11-1.1.1 LIVY_HOME=/home/admin/apache-livy-0.6.0-incubating-bin KYLIN_VERSION=3.0.1 KYLIN_HOME=/home/admin/apache-kylin-3.0.1-bin-hbase1x SQOOP_HOME=/home/admin/sqoop-1.4.7.bin__hadoop-2.6.0]

[root@c8-2 ~]# docker images
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
apachekylin/apache-kylin-standalone             3.0.1               22b22aeded23        5 minutes ago       4.62GB
docker.elastic.co/elasticsearch/elasticsearch   7.8.0               121454ddad72        2 weeks ago         810MB
apachekylin/apache-kylin-standalone             <none>              2cd68ccd1e5d        4 months ago        2.49GB

[root@c8-2 ~]# docker ps -a
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                NAMES
388e9b6f5a75        docker.elastic.co/elasticsearch/elasticsearch:7.8.0   "/tini -- /usr/local…"   3 hours ago         Up 3 hours          0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                                                                                                       elasticsearch
070dd7124d70        2cd68ccd1e5d                                          "/home/admin/entrypo…"   8 days ago          Up 3 hours          0.0.0.0:7070->7070/tcp, 0.0.0.0:8032->8032/tcp, 0.0.0.0:8042->8042/tcp, 0.0.0.0:8088->8088/tcp, 0.0.0.0:16010->16010/tcp, 0.0.0.0:50070->50070/tcp   kylin

[root@c8-2 ~]# docker history 2cd68ccd1e5d
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
2cd68ccd1e5d        4 months ago        /bin/sh -c #(nop)  ENTRYPOINT ["/home/admin/…   0B                  
<missing>           4 months ago        /bin/sh -c chmod u+x /home/admin/entrypoint.…   2.97kB              
<missing>           4 months ago        /bin/sh -c #(nop) COPY file:965b85e80a7599f1…   2.97kB              
...


docker run -d --name kylin_orig -m 8G -p 7070:7070 -p 8088:8088 -p 50070:50070 -p 8032:8032 -p 8042:8042 -p 16010:16010 apachekylin/apache-kylin-standalone:3.0.1
```