---
title: "Hướng dẫn cài đặt Data Warehouse trên Hadoop (phần 1)"
layout: post
date: 2023-01-07 09:00:00 +0700
image: /assets/images/blog/bigdata/2023-01-07/data_warehouse.jpeg
headerImage: false
tag:
- bigdata
category: blog
author: Long Nguyen
description: 
---

# Nội dung

1. [Cài đặt Spark](#install_spark)
2. [Cài đặt Postgresql và Hive](#install_hive)

## Cài đặt Spark <a name="install_spark"></a>

Bạn lên trang chủ của Spark [tại đây](download_spark) để lấy link download. Vào thời điểm viết bài này phiên bản spark mới nhất là 3.3.1, tuy nhiên khi thử nghiệm mình thấy phiên bản này không tương tích với DBT và Hive nên mình sử dụng phiên bản spark thấp hơn là 3.1.1. 

> Lưu ý: Do đã có sẵn cụm Hadoop rồi nên chúng ta chỉ cần cài Spark trên 1 node (mình cái trên `node01`), khi chạy job Spark ta để cấu hình `--master yarn` thì job sẽ được chạy được trên tất cả các node.

```sh
$ wget https://archive.apache.org/dist/spark/spark-3.1.1/spark-3.1.1-bin-hadoop3.2.tgz
$ tar -xzvf spark-3.1.1-bin-hadoop3.2.tgz 
$ mv spark-3.1.1-bin-hadoop3.2 /lib/spark
$ mkdir /lib/spark/logs
$ chgrp hadoop -R /lib/spark
$ chmod g+w -R /lib/spark
```

Cấu hình các biến môi trường trong file `/etc/bash.bashrc`:

```sh
export SPARK_HOME=/lib/spark
export PATH=$PATH:$SPARK_HOME/bin
```

Cập nhật biến môi trường

```sh
$ source /etc/bash.bashrc
```

Tạo file `$SPARK_HOME/conf/spark-env.sh`:

```sh
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
```

Thêm cấu hình classpath vào file `$SPARK_HOME/conf/spark-env.sh` vừa tạo:

```sh
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

Kiểm tra xem đã chạy được Spark ở yarn mode hay chưa:

```sh
spark-shell --master yarn --deploy-mode client
```

Kết quả:
```sh
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.1.1
      /_/
         
Using Scala version 2.12.10 (OpenJDK 64-Bit Server VM, Java 11.0.17)
scala> 
```

## Cài đặt Postgresql <a name="install_postgresql"></a>

Bạn có thể cài đặt trực tiếp Posgresql hoặc sử dụng Docker, ở đây mình sẽ cài trực tiếp trên `node01` theo hướng dẫn trên trang chủ [tại đây](install_postgresql).

Bật service postgresql

```sh
$ service postgresql start
```

Theo mặc định ta chỉ có thể connect vào Postgresql trên máy localhost thông qua user `postgres`:

Chuyển qua user postgres
```sh
$ su postgres
```

Vào giao diện Sql command:
```sh
$ psql
```

Set lại mật khẩu cho account postgres:

```sql
postgres=# ALTER USER postgres WITH PASSWORD 'password';
```

Để có thể connect từ các máy khác (remote) ta cần sửa lại cấu hình như sau:

- Chỉnh sửa cấu hình trong: `postgresql.conf`
```sh 
listen_addresses = '*'
```

- Bổ sung cấu hình sau vào cuối file: `pg_hba.conf`
```
host all all 0.0.0.0/0 md5
```

> Lưu ý: để biết vị trí của 2 file cấu hình ta cần  dùng lệnh `show config_file;` và `show hba_file;` trên Sql command.

Restart posgresql
```sh
$ service postgresql restart
```

Kiểm tra xem đã connect được vào postgresql thông qua ip chưa:

```sh
$ psql -h node01 -p 5432 -U postgres w
```

## Cấu hình Spark Thrift Server (Hive) <a name="install_hive"></a>

Trong bản cài đặt của Spark đã có tích hợp sẵn Thrift Server (Hive), cho phép các ứng dụng khác có thể làm việc với Spark thông qua ngôn ngữ SQL của Hive. Ta cần cấu hình để dữ liệu được lưu trữ trên HDFS và metatadata lưu trữ trên postgresql như sau:

- `$SPARK_HOME/conf/hive-site.xml`
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>hive.metastore.db.type</name>
        <value>postgres</value>
    </property>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>hdfs://node01:9000/user/hive/warehouse</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:postgresql://node01:5432/metastore</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.postgresql.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>postgres</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>password</value>
    </property>

    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <property>
       <name>fs.hdfs.impl.disable.cache</name>
       <value>true</value>
    </property>

</configuration>
```

- `$SPARK_HOME/sbin/start-hive.sh`
```sh
./start-thriftserver.sh --master yarn --deploy-mode client \
    --driver-memory 4g \
    --executor-memory 6g \
    --executor-cores 4 \
    --num-executors 2 \
    --packages org.apache.hudi:hudi-spark3-bundle_2.12:0.10.0 \
    --conf spark.hadoop.datanucleus.autoCreateTables=true \
    --conf spark.hadoop.datanucleus.schema.autoCreateTables=true \
    --conf spark.hadoop.datanucleus.fixedDatastore=false \
    --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
    --conf spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension \
    --conf spark.driver.userClassPathFirst=true
```

> Thrift server sẽ chạy như một Spark Job trên Yarn vì thế bạn có thể tùy chỉnh tài nguyên phù hợp (dung lượng ram, số nhân, số excurtors...) khi chạy.

Download driver postgresql vào thư mục `$SPARK_HOME/jars/`:
```sh
$ cd $SPARK_HOME/jars/
$ wget wget https://jdbc.postgresql.org/download/postgresql-42.5.1.jar
```

Tạo và phân quyền cho thư mục `warehouse` trên HDFS:
```sh
$ hdfs dfs -mkdir -p /user/hive/warehouse
$ hdfs dfs -chmod -R 774 /user/hive/warehouse
```

Tạo database `metastore` trên Postgresql:
```sql
postgres=# CREATE DATABASE metastore;
```

Chạy Thrift Server
```sh
$ chmod +x $SPARK_HOME/sbin/start-hive.sh
$ $SPARK_HOME/sbin/start-hive.sh
```

Job đã chạy trên Yarn:
- `http://172.24.0.2:8088/cluster/scheduler`
![Thrift Server](/assets/images/blog/bigdata/2023-01-07/thrift_run_on_yarn.png)


Tắt Thift Server:
```sh
$ $SPARK_HOME/sbin/stop-thriftserver.sh
```

## Cài đặt DBT <a name="install_dbt"></a>

Để cho nhanh thì mình sẽ sử dụng project example của DBT có sẵn trên Github [tại đây](https://github.com/dbt-labs/jaffle_shop). Mình sẽ giải thích kỹ hơn về project này trong bài viết sau.

Sử dụng git để clone project về máy:
```sh
$ git clone https://github.com/dbt-labs/jaffle_shop.git
```

Cài đặt môi trường phát triển và các thư viện qua Anaconda (bạn có thể tham khảo về Anaconda [tại đây](https://www.anaconda.com/))

```sh
$ conda create -n dbt_example python=3.9
$ conda activate dbt_example
(dbt_example) $ pip install dbt-spark[PyHive]
```

Cấu hình connect đến Hive server trong file `~/dbt/profiles.yml`:
```yml
jaffle_shop:
  outputs:
    dev:
      type: spark
      method: thrift
      host: 172.24.0.2
      port: 10000
      user: postgres
      dbname: jaffle_shop
      schema: dbt_alice
      threads: 4

    prod:
      type: spark
      method: thrift
      host: 172.24.0.2
      port: 10000
      user: postgres
      dbname: jaffle_shop
      schema: dbt_alice
      threads: 4

  target: dev
```

Lần lượt chạy các lệnh sau để migrate các model thành các bảng trên DWH

```sh
$ dbt debug
$ dbt seed
$ dbt run
$ dbt test
$ dbt docs generate
$ dbt docs serve
```

Kiểm tra project DBT đã chạy trên `http://localhost:8080/`
![DBT Screen](/assets/images/blog/bigdata/2023-01-07/dbt_screen.png)

Kiểm tra thư mục `warehouse` trên HDFS: `http://172.24.0.2:9870/explorer.html#/user/hive/warehouse` ta có thể thấy có các database dữ liệu đã được tạo ra:
![DBT HDFS](/assets/images/blog/bigdata/2023-01-07/dbt_hdfs.png)

## Cài đặt Superset <a name="install_superset"></a>

Chúng ta sẽ sử dụng Superset để xem dữ liệu được migrate trên trên DWH. Mình sẽ cài Superset thông qua docker

```sh
docker run -d --name superset --hostname superset --network hadoop apache/superset
docker exec -it superset superset fab create-admin \
              --username admin \
              --firstname Superset \
              --lastname Admin \
              --email admin@superset.com \
              --password admin
```

Kiểm tra trên giao diện web của Superset: `http://172.24.0.4:8088/`

![Superset Screen](/assets/images/blog/bigdata/2023-01-07/superset_screen.png)

Đăng nhập bằng account `admin/admin`, sau đó vào `Settting \ Databases Connections` để tạo một Connection Database mới. Trong step 1 bạn chọn Supported Database là Apache Hive, trong phần SQLALCHEMY URI bạn điền url của Hive: `hive://postgres@172.24.0.2:10000/jaffle_shop` sau đó chọn Connect.

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2023-01-07/superset_connect.png" alt="Superset Connect" width="350"></p>

Để sử dụng các truy vấn vào DWH, bạn sử dụng giao diện SQL Lab trên Superset:

![SQL Lab](/assets/images/blog/bigdata/2023-01-07/sql_lab.png)

## Cài đặt 

[download_spark]: https://spark.apache.org/downloads.html
[install_postgresql]: https://www.postgresql.org/download/linux/ubuntu/