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
2. [Cài đặt Hive](#install_hive)

## Cài đặt Spark <a name="install_spark"></a>

Bạn lên trang chủ của Spark [tại đây](download_spark) để download bản mới nhất.

```sh
$ wget https://dlcdn.apache.org/spark/spark-3.3.1/spark-3.3.1-bin-hadoop.tgz
$ tar -xvzf spark-3.3.1-bin-hadoop.tgz
$ mv spark-3.3.1-bin-hadoop /lib/spark
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
   /___/ .__/\_,_/_/ /_/\_\   version 3.3.1
      /_/
         
Using Scala version 2.12.15 (OpenJDK 64-Bit Server VM, Java 11.0.16)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```

## Cài đặt Hive <a name="install_hive"></a>

[download_spark]: https://spark.apache.org/downloads.html