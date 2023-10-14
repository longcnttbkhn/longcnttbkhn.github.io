---
title: "DWH 3: Cài đặt Trino truy vấn dữ liệu trong Data warehouse"
layout: post
date: 2023-10-09 09:00:00 +0700
image: /assets/images/blog/bigdata/2023-10-09/trino_ui.jpg
headerImage: false
tag:
- bigdata
category: blog
author: Long Nguyen
description: Trong bài viết này mình sẽ hướng dẫn các bạn cách cài đặt và cấu hình Trino làm query engine trong Data warehouse thay thế cho Spark Thrift Server.
---

Trong bài viết trước mình đã giới thiệu với các bạn một thiết kế và cài đặt Data Warehouse dựa trên nền tảng Hadoop và một số công nghệ Opensource, bạn có thể xem lại [tại đây](/cai-dat-data-warehouse-tren-hadoop-phan-1/). Thiết kế này sử dụng Spark Thrift Server (STS) là một JDBC/ODBC server cho phép các ứng dụng BI có thể connect vào DWH để yêu cầu thực thi các truy vấn SQL. Tuy nhiên STS khi khởi chạy chỉ tạo ra ra 1 đối tượng Spark Context do đó tại mỗi thời điểm chỉ có 1 truy vấn có thể được thực thi. Để giải quyết vấn đề này thì mình sử dụng Trino làm query engine, nó giúp điều phối tài nguyên hợp lý hơn và chạy được nhiều truy vấn đồng thời. 

# Nội dung
1. [Giới thiệu tổng quan](#introduction)
2. [Cài đặt Hive Metastore (HMS)](#install-hms)
3. [Cài đặt và cấu hình Trino trên 1 node](#install-trino)
4. [Thêm node mới vào cụm Trino](#add-node)
5. [Kết luận](#conclusion)

## Giới thiệu tổng quan <a name="introduction"></a>

[Trino](https://trino.io/) là một SQL query engine phân tán hiệu năng cao dùng cho các hoạt động phân tích dữ liệu lớn, có thể làm việc với rất nhiều datasource khác nhau thông qua các connector, từ đó cho phép việc truy vấn kết hợp nhiều datasource. Trino không phải là Database cũng không phải là Datawarehouse, nó không lưu trữ dữ liệu, nó chỉ cung cấp một giao diện SQL cho người dùng và các ứng dụng khác truy vấn dữ liệu trên các Datasource khác nhau bằng ngôn ngữ SQL. Trino được xây dựng để phù hợp với các tác vụ Online Analyst Processing (OLAP) thay vì Online Transaction Processing (OLTP).

Một số khái niệm trong Trino:
- *Datasource*: Là một nguồn dữ liệu mà Trino sử dụng, nó có thể là database, hệ thống file (local, hdfs, s3...), google sheet excel, elasticsearch, thậm chí cả kafka... Trino sử dụng connector tương ứng với từng datasource để kết nối với chúng.
- *Connector*: Đóng vai trò như người phiên dịch giữa Trino và các Datasource, bạn có thể xem danh sách các datasource đầy đủ [tại đây](https://trino.io/docs/current/connector.html)
- *Catalog*: Là một khai báo về một Datasource trong Trino, một khai báo catalog bao gồm tên connector và các cấu hình thích hợp để Trino có thể sử dụng Datasource. Trong SQL thì catalog là cấp đầu tiên khi cần gọi đến 1 table: \<catalog\>.\<schema\>.\<table\>
- *Schema*: là một nhóm được đặt tên gồm nhiều bảng trong catalog, nó tương đương với khái niệm database trong các cơ sở dữ liệu quan hệ.
- *Table*: là một bảng chứa dữ liệu, sử dụng SQL người dùng có thể tạo, thay đổi, truy vấn hoặc thêm dữ liệu vào bảng.

Trino là một query engine phân tán tức là nó có thể được cài đặt trên nhiều node và kết hợp cùng nhau thành 1 cụm (cluster). Cụm Trino cũng được thiết kế theo kiến trúc master-slaves, có một node làm master đóng vài trò quản lý, điều phối, lập lịch cho cả cụm và các node slave đóng vai trò thực thi các nhiệm vụ được giao bởi master. Các thành phần trong cụm Trino bao gồm:
- *Coordinator*: Đóng vai trò master trong cụm Trino, là server để nhận truy vấn SQL từ client, phân tích cú pháp, lên kế hoạch thực thi và giao nhiệm vụ cho các Worker, tổng hợp kết quả từ các worker và trả về cho client.
- *Worker*: Đóng vai trò là node slave trong cụm Trino, nó tiếp nhận và xử lý nhiệm vụ được giao bởi Coordinator.

## Cài đặt Hive Metastore (HMS) <a name="install-hms"></a>

Trước khi bắt đầu thì mình sẽ giải thích một chút về lý do vì sao chúng ta lại cần HMS. Nếu các bạn còn nhớ, trong bài viết trước khi mình cài Data warehouse (bạn có thể xem lại [tại đây](/cai-dat-data-warehouse-tren-hadoop-phan-1/#install_hive)) mình đã làm 2 việc:
- Một là tạo và cấu hình thư mục lưu trữ dữ liệu trong DWH, một thư mục trên HDFS: `hdfs://node01:9000/user/hive/warehouse`
- Hai là tạo và hình nơi lưu trữ metadata cho DWH, một cơ sở dữ liệu trong postgresql: `jdbc:postgresql://node01:5432/metastore` 

Đây chính là 2 thành phần để tạo nên một Data warehouse là *Storage* và *Metadata*. Trong đó Storage là nơi dữ liệu được lưu trữ còn Metadata là nơi lưu trữ các thông tin như các schema (database), các table, cấu trúc của table, loại table, nơi lưu trữ dữ liệu của table trong Storage... Chỉ cần có 2 thành phần này thì một query engine đã có thể truy vấn dữ liệu trong DWH, trong bài viết trước mình đã sử dụng Spark Thrift Server để làm query engine server, tuy nhiên cũng có thể thao tác với DWH bằng Spark SQL (xem thêm [tại đây](https://spark.apache.org/docs/latest/sql-distributed-sql-engine.html)) hoặc trong một spark job (xem thêm [tại đây](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html))

Để truy vấn được data trong DWH thì Trino cần 1 service cung cấp các thông tin về Storage và Metadata, đây chính là việc mà HMS sẽ làm.

Lý thuyết thế thôi giờ chúng ta bắt tay vào cài đặt nhé, mình sẽ thực hiện trên cụm DWH đã có trong bài viết trước, bạn có thể xem lại [tại đây](/cai-dat-data-warehouse-tren-hadoop-phan-1). Đầu tiên chúng ta sẽ lên trang chủ của Hive để download phiên bản hive phù hợp [tại đây](https://dlcdn.apache.org/hive/) mình sẽ sử dụng phiên bản 2.3.9 do đây là phiên bản mà Spark Thrift Server của mình đang sử dụng.
> Lưu ý: Muốn biết STS đang dùng Hive version nào bạn chạy job STS lên sau đó vào giao diện SparkUI của job, vào thẻ Environment và tìm cấu hình `spark.sql.hive.version`

```sh
$ wget https://dlcdn.apache.org/hive/hive-2.3.9/apache-hive-2.3.9-bin.tar.gz
$ tar -xvzf apache-hive-2.3.9-bin.tar.gz
$ mv apache-hive-2.3.9-bin /lib/hive
$ mkdir /lib/hive/logs
$ chgrp hadoop -R /lib/hive
$ chmod g+w -R /lib/hive
```

Bổ sung biến môi trường `/etc/bash.bashrc`

```sh
export HIVE_HOME=/lib/hive
export PATH=$HIVE_HOME/bin:$PATH
```
Cập nhật biến môi trường

```sh
$ source /etc/bash.bashrc
```

Copy cấu hình và thư viện từ Spark sang

```sh
$ cp $SPARK_HOME/conf/hive-site.xml $HIVE_HOME/conf/
$ cp $SPARK_HOME/jars/postgresql-42.5.1.jar $HIVE_HOME/lib/
```

Run HMS trên user hive

```sh
[hive]$ hive --service metastore &
```

Mặc định HMS sẽ được chạy trên port `9083` bạn có thể thay đổi nó bằng cấu hình `hive.metastore.port` trong file `$HIVE_HOME/conf/hive-site.xml`

## Cài đặt và cấu hình Trino trên 1 node <a name="install-trino"></a>

Bạn lên trang chủ của Trino để [tại đây](https://trino.io/docs/current/release.html) tìm phiên bản phù hợp cho hệ thống, mình lựa chọn phiên bản Trino 389 do đây là phiên bản mới nhất còn hỗ trợ java 11.
> Lưu ý: bạn có thể update java lên phiên bản mới nhất để sử dụng Trino latest version theo hướng dẫn [tại đây](https://trino.io/docs/current/installation/deployment.html)

```sh
$ wget https://repo1.maven.org/maven2/io/trino/trino-server/389/trino-server-389.tar.gz
$ tar -xvzf trino-server-389.tar.gz
$ mv trino-server-389 /lib/trino
$ mkdir /lib/trino/logs
$ chgrp hadoop -R /lib/trino
$ chmod g+w -R /lib/trino
```

Bổ sung biến môi trường `/etc/bash.bashrc`

```sh
export TRINO_HOME=/lib/trino
export PATH=$TRINO_HOME/bin:$PATH
```
Cập nhật biến môi trường

```sh
$ source /etc/bash.bashrc
```

Tạo lần lượt các file cấu hình như sau:

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

> Lưu ý: Ở đây mình sử dụng connector `delta-lake` để Trino có thể làm việc với Delta table

Cài đặt python

```sh
$ ln -s /usr/bin/python3 /usr/bin/python
```

Tạo user trino

```sh
$ useradd -g hadoop -m -s /bin/bash trino
```

Run Trino trên user trino

```sh
[trino]$ launcher start
```

Kiểm tra trạng thái của Trino

```sh
[trino]$ launcher status
```

Xem giao diện TrinoUI: `http://node01:8080/ui/`

![Trino UI](/assets/images/blog/bigdata/2023-10-09/trino_ui.jpg)

Cài đặt Trino CLI

```sh
$ wget https://repo1.maven.org/maven2/io/trino/trino-cli/389/trino-cli-389-executable.jar
$ mv trino-cli-389-executable.jar $TRINO_HOME/bin/trino
$ chmod +x $TRINO_HOME/bin/trino
$ trino http://node01:8080/hive
```

Chạy một số lệnh trên Trino CLI

```sql
trino> show catalogs;
trino> show schemas;
trino> select * from ...
```

Tắt Trino

```sh
[trino]$ launcher stop
```

## Thêm node mới vào cụm Trino <a name="install-trino"></a>

Để thêm node mới vào cụm Trino đang có, bạn lặp lại các bước ở trên và sửa lại 2 file cấu hình:
 
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

## Kết luận <a name="conclusion"></a>

Trong bài viết này mình đã hướng dẫn cách cài đặt Trino làm query engine server để thay thế cho Spark Thrift Server, chúc các bạn thành công!