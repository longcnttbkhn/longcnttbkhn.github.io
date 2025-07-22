---
title: "Using Trino to query data from Data Warehouse (Part 3)"
layout: post
date: 2023-10-09 10:00:00 +0700
image: /assets/images/blog/bigdata/2023-10-09/trino_ui.jpg
headerImage: false
tag:
- bigdata
category: english
author: Lake Nguyen
description: In this article, I will guide you how to install and configure Trino as a query engine in Data warehouse to replace Spark Thrift Server.
---

In the previous article, I introduced to you a design and installation of Data Warehouse based on Hadoop platform and some Opensource technologies, you can review [here](/how-to-build-data-warehouse-on-hadoop-cluster-part-1/). This design uses Spark Thrift Server (STS) which is a JDBC/ODBC server that allows BI applications to connect to DWH to request execution of SQL queries. However, when launched, STS only creates 1 Spark Context object, so at each time only 1 query can be executed. To solve this problem, I use Trino as a query engine, it helps to coordinate resources more reasonably and run multiple queries at the same time.

# Contents
1. [Overview](#introduction)
2. [Install Hive Metastore (HMS)](#install-hms)
3. [Install and configure Trino on a node](#install-trino)
4. [Add new node to Trino cluster](#add-node)
5. [Conclusion](#conclusion)

## Overview <a name="introduction"></a>

[Trino](https://trino.io/) is a high-performance distributed SQL query engine for big data analysis, which can work with many different datasources through connectors, thereby allowing queries to combine multiple datasources. Trino is neither a Database nor a Datawarehouse, it does not store data, it only provides an SQL interface for users and other applications to query data on different Datasources using SQL language. Trino is built to suit Online Analyst Processing (OLAP) tasks instead of Online Transaction Processing (OLTP).

Some concepts in Trino:

- *Datasource*: Is a data source that Trino uses, it can be a database, file system (local, hdfs, s3...), google sheet excel, elasticsearch, even kafka... Trino uses connectors corresponding to each datasource to connect to them.
- *Connector*: Acts as a translator between Trino and the Datasources, you can see the full list of datasources [here](https://trino.io/docs/current/connector.html)
- *Catalog*: Is a declaration of a Datasource in Trino, a catalog declaration includes the connector name and the appropriate configurations for Trino to use the Datasource. In SQL, the catalog is the first level when calling a table: \<catalog\>.\<schema\>.\<table\>
- *Schema*: is a named group of multiple tables in the catalog, it is equivalent to the concept of a database in relational databases.
- *Table*: is a table containing data, using SQL users can create, change, query or add data to the table.

Trino is a distributed query engine, meaning it can be installed on multiple nodes and combined together into a cluster. The Trino cluster is also designed according to the master-slaves architecture, with a master node that plays the role of managing, coordinating, and scheduling the entire cluster and slave nodes that play the role of executing tasks assigned by the master. The components in the Trino cluster include:
- *Coordinator*: Acts as the master in the Trino cluster, is the server to receive SQL queries from the client, parse the syntax, plan the execution and assign tasks to the Workers, synthesize the results from the workers and return them to the client.
- *Worker*: Acts as the slave node in the Trino cluster, it receives and processes tasks assigned by the Coordinator.

## Install Hive Metastore (HMS) <a name="install-hms"></a>

Before we start, I will explain a little bit about why we need HMS. If you remember, in the previous article when I installed Data warehouse (you can review [here](/cai-dat-data-warehouse-tren-hadoop-phan-1/#install_hive)) I did 2 things:
- One is to create and configure the data storage directory in DWH, a directory on HDFS: `hdfs://node01:9000/user/hive/warehouse`
- Two is to create and configure the metadata storage location for DWH, a database in postgresql: `jdbc:postgresql://node01:5432/metastore`

These are the 2 components to create a Data warehouse: *Storage* and *Metadata*. In which Storage is where data is stored and Metadata is where information such as schemas (databases), tables, table structures, table types, table data storage locations in Storage are stored... With just these two components, a query engine can query data in DWH. In the previous article, I used Spark Thrift Server as a query engine server, but it is also possible to manipulate DWH using Spark SQL (see more [here](https://spark.apache.org/docs/latest/sql-distributed-sql-engine.html)) or in a spark job (see more [here](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html))

To query data in DWH, Trino needs a service that provides information about Storage and Metadata, this is what HMS will do.

That's the theory, now let's start installing, I will do it on the DWH cluster in the previous article, you can review [here](/how-to-build-data-warehouse-on-hadoop-cluster-part-1/). First, we will go to the Hive homepage to download the appropriate hive version [here](https://dlcdn.apache.org/hive/), I use version 2.3.9 because this is the version that my Spark Thrift Server is using.
> Note: To know which Hive version STS is using, run the STS job, then go to the job's SparkUI interface, go to the Environment tab and find the configuration `spark.sql.hive.version`

```sh
$ wget https://dlcdn.apache.org/hive/hive-2.3.9/apache-hive-2.3.9-bin.tar.gz
$ tar -xvzf apache-hive-2.3.9-bin.tar.gz
$ mv apache-hive-2.3.9-bin /lib/hive
$ mkdir /lib/hive/logs
$ chgrp hadoop -R /lib/hive
$ chmod g+w -R /lib/hive
```

Add environment variables `/etc/bash.bashrc`

```sh
export HIVE_HOME=/lib/hive
export PATH=$HIVE_HOME/bin:$PATH
```

Update environment variables

```sh
$ source /etc/bash.bashrc
```

Copy configuration and libraries from Spark

```sh
$ cp $SPARK_HOME/conf/hive-site.xml $HIVE_HOME/conf/
$ cp $SPARK_HOME/jars/postgresql-42.5.1.jar $HIVE_HOME/lib/
```

Run HMS on user hive

```sh
[hive]$ hive --service metastore &
```

By default, HMS will run on port `9083`, you can change it by configuring `hive.metastore.port` in the file `$HIVE_HOME/conf/hive-site.xml`

## Install and configure Trino on a node <a name="install-trino"></a>

Go to Trino's homepage to [here](https://trino.io/docs/current/release.html) find the appropriate version for your system, I choose Trino version 389 because this is the latest version that still supports java 11.
> Note: you can update java to the latest version to use Trino latest version according to the instructions [here](https://trino.io/docs/current/installation/deployment.html)

```sh
$ wget https://repo1.maven.org/maven2/io/trino/trino-server/389/trino-server-389.tar.gz
$ tar -xvzf trino-server-389.tar.gz
$ mv trino-server-389 /lib/trino
$ mkdir /lib/trino/logs
$ chgrp hadoop -R /lib/trino
$ chmod g+w -R /lib/trino
```

Add environment variables `/etc/bash.bashrc`

```sh
export TRINO_HOME=/lib/trino
export PATH=$TRINO_HOME/bin:$PATH
```
Update environment variables

```sh
$ source /etc/bash.bashrc
```

Create the configuration files in turn as follows:

- `$TRINO_HOME/etc/config.properties`

```sh
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=8080
discovery.uri=http://node01:8080
```

- `$TRINO_HOME/etc/jvm.config`

```sh
-server
-Xmx16G
-XX:InitialRAMPercentage=80
-XX:MaxRAMPercentage=80
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExitOnOutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-XX:ReservedCodeCacheSize=512M
-XX:PerMethodRecompilationCutoff=10000
-XX:PerBytecodeRecompilationCutoff=10000
-Djdk.attach.allowAttachSelf=true
-Djdk.nio.maxCachedBufferSize=2000000
-XX:+UnlockDiagnosticVMOptions
-XX:+UseAESCTRIntrinsics
-Dfile.encoding=UTF-8
# Disable Preventive GC for performance reasons (JDK-8293861)
#-XX:-G1UsePreventiveGC  
# Reduce starvation of threads by GClocker, recommend to set about the number of cpu cores (JDK-8192647)
-XX:GCLockerRetryAllocationCount=32
```

- `$TRINO_HOME/etc/log.properties`

```sh
io.trino=INFO
```

- `$TRINO_HOME/etc/node.properties`

```sh
node.environment=production
node.id=node01
node.data-dir=/home/trino/data
```

- `$TRINO_HOME/etc/catalog/hive.properties`

```sh
connector.name=delta-lake
hive.metastore.uri=thrift://node1:9083
hive.metastore-cache.cache-partitions=false
```

> Note: Here I use the `delta-lake` connector so that Trino can work with Delta table

Install python

```sh
$ ln -s /usr/bin/python3 /usr/bin/python
```

Create user trino

```sh
$ useradd -g hadoop -m -s /bin/bash trino
```

Run Trino as user trino

```sh
[trino]$ launcher start
```

Check Trino status

```sh
[trino]$ launcher status
```

Check TrinoUI: `http://node01:8080/ui/`

![Trino UI](/assets/images/blog/bigdata/2023-10-09/trino_ui.jpg)

Install Trino CLI

```sh
$ wget https://repo1.maven.org/maven2/io/trino/trino-cli/389/trino-cli-389-executable.jar
$ mv trino-cli-389-executable.jar $TRINO_HOME/bin/trino
$ chmod +x $TRINO_HOME/bin/trino
$ trino http://node01:8080/hive
```

Run some commands on Trino CLI

```sql
trino> show catalogs;
trino> show schemas;
trino> select * from ...
```

Stop Trino

```sh
[trino]$ launcher stop
```

## Add new node to Trino cluster <a name="install-trino"></a>

To add a new node to an existing Trino cluster, repeat the steps above and edit the 2 configuration files:
 
- `$TRINO_HOME/etc/config.properties``

```sh
coordinator=false
http-server.http.port=8080
discovery.uri=http://node01:8080
```

- `$TRINO_HOME/etc/node.properties`

```sh
node.environment=production
node.id=node02
node.data-dir=/home/trino/data
```

## Conclusion <a name="conclusion"></a>

In this article, I have guided you on how to install Trino as a query engine server to replace Spark Thrift Server, I wish you success!