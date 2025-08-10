---
title: "Hướng dẫn cài Hadoop cluster trên ubuntu 20.04"
layout: post
date: 2022-10-01 09:27:10 +0700
image: /assets/images/blog/bigdata/2022-10-01/hadoop-logo.jpeg
headerImage: false
tag:
- bigdata
category: blog
author: Long Nguyen
description: Trong bài viết này mình sẽ mô tả lại chi tiết quá trình cài đặt, cấu hình Hadoop trên 3 node giả lập bằng Docker. Mình cài Hadoop bản mới nhất (3.3.4) trên hệ điều hành Ubuntu 20.04 và Java 11.
---

[Apache Hadoop][apache-hadoop] là một dự án phần mềm nguồn mở được sử dụng để xây dựng các hệ thống xử lý dữ liệu lớn, cho phép tính toán phân tán và mở rộng trên các cụm tới hàng ngàn máy tính với khả năng sẵn sàng và chịu lỗi cao. Hiện nay Hadoop đã phát triển trở thành một hệ sinh thái với rất nhiều sản phẩm, dịch vụ khác nhau. Trước đây mình sử dụng [Ambari HDP][hdp] để cài đặt và quản lý Hadoop Ecosystem, công cụ này cho phép tập trung tất cả cấu hình của các dịch vụ Hadoop về một nơi, từ đó dễ dàng quản lý và mở rộng node khi cần. Tuy nhiên từ năm 2021 HDP đã đóng lại để thu phí, tất cả các repository đều yêu cầu tài khoản trả phí để có thể download và cài đặt. Gần đây mình có nhu cầu cần cài đặt hệ thống Hadoop mới, mình quyết định cài tay từng thành phần, tuy sẽ phức tạp và tốn nhiều công sức hơn nhưng mình có thể kiểm soát dễ dàng hơn không bị phụ thuộc vào bên khác, một phần cũng do hệ thống mới chỉ có 3 node nên khối lượng công việc cũng không bị thêm quá nhiều. Toàn bộ quá trình cài đặt mình sẽ ghi chép lại chi tiết trong series các bài viết thuộc chủ đề `bigdata` mọi người chú ý đón đọc nhé!

## Nội dung
1. [Mục tiêu](#target)
2. [Cài đặt môi trường](#environment)
3. [Download hadoop và cấu hình](#config)
4. [Chạy trên 1 node](#run-single-node)
5. [Thêm node mới vào cụm](#add-node)
6. [Hướng dẫn sử dụng cơ bản](#user-guide)
7. [Kết luận](#conclusion)


## Mục tiêu <a name="target"></a>

Trong bài viết này mình sẽ cài đặt Hadoop bản mới nhất (3.3.4 vào thời điểm viết bài này) trên 3 node Ubuntu 20.04 và OpenJdk11. Để thuận tiện cho việc setup và thử nghiệm mình sẽ sử dụng Docker để giả lập 3 node này.

## Cài đặt môi trường <a name="environment"></a>

Đầu tiên chúng ta tạo một bridge network mới trên Docker (Nếu chưa cài Docker các bạn xem hướng dẫn cài [tại đây][docker])

```sh
$ docker network create hadoop 
```

Tiếp theo là tạo một container trên image Ubuntu 20.04

```sh
$ docker run -it --name node01 -p 9870:9870 -p 8088:8088 -p 19888:19888 --hostname node01 --network hadoop ubuntu:20.04
```

> Mình đang sử dụng MacOS nên cần binding port từ container ra máy host, bạn không cần làm điều này nếu sử dụng Linux hoặc Window.

Cài đặt các package cần thiết

```sh
$ apt update
$ apt install -y wget tar ssh default-jdk 
```

Tạo user hadoop

```sh
$ groupadd hadoop
$ useradd -g hadoop -m -s /bin/bash hdfs
$ useradd -g hadoop -m -s /bin/bash yarn
$ useradd -g hadoop -m -s /bin/bash mapred
```

> Vì lý do bảo mật, Hadoop khuyến nghị mỗi dịch vụ nên chạy trên một user khác nhau, xem chi tiết [tại đây][hadoop-secure]

Tạo ssh-key trên mỗi user

```sh
$ su <username>
$ ssh-keygen -m PEM -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

Start ssh service

```sh
$ service ssh start
```

Thêm hostname trong file `/etc/hosts`

```sh
172.20.0.2      node01
```

> Lưu ý `172.20.0.2` là ip container trên máy của mình, bạn thay bằng ip máy của bạn.

Kiểm tra xem đã ssh được vào hay chưa

```sh
$ ssh <username>@node01
```

---

## Download hadoop và cấu hình <a name="config"></a>

Ta lên trang chủ download của Hadoop [tại đây][download-hadoop] để lấy link down bản mới nhất.

```sh
$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz
$ tar -xvzf hadoop-3.3.4.tar.gz
$ mv hadoop-3.3.4 /lib/hadoop
$ mkdir /lib/hadoop/logs
$ chgrp hadoop -R /lib/hadoop
$ chmod g+w -R /lib/hadoop
```

Tiếp theo cần cấu hình biến môi trường, ở đây chúng ta sẽ thêm các biến môi trường vào file `/etc/bash.bashrc` để tất cả các user trên hệ thống đều có thể sử dụng

```sh
export JAVA_HOME=/usr/lib/jvm/default-java
export HADOOP_HOME=/lib/hadoop
export PATH=$PATH:$HADOOP_HOME/bin

export HDFS_NAMENODE_USER="hdfs"
export HDFS_DATANODE_USER="hdfs"
export HDFS_SECONDARYNAMENODE_USER="hdfs"
export YARN_RESOURCEMANAGER_USER="yarn"
export YARN_NODEMANAGER_USER="yarn"

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

Cập nhật biến môi trường

```sh
$ source /etc/bash.bashrc
```

Cũng cần cập nhật biến môi trường trong file: `$HADOOP_HOME/etc/hadoop/hadoop-env.sh`

```sh
export JAVA_HOME=/usr/lib/jvm/default-java
```

Thiết lập cấu hình cho Hadoop
- `$HADOOP_HOME/etc/hadoop/core-site.xml` xem full cấu hình [tại đây][core-site-default]

{% highlight xml %}
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/${user.name}/hadoop</value>
    </property>
</configuration>
{% endhighlight %}

> `/home/${user.name}/hadoop` là thư mục mình lưu dữ liệu trên HDFS, bạn có thể đổi sang thư mục khác nếu muốn.

- `$HADOOP_HOME/etc/hadoop/hdfs-site.xml` xem full cấu hình [tại đây][hdfs-site]

{% highlight xml %}
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.permissions.superusergroup</name>
        <value>hadoop</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir.perm</name>
        <value>774</value>
    </property>
</configuration>
{% endhighlight %}

> Cấu hình `dfs.replication` thiết lập số bản sao thực tế được lưu trữ đối với một dữ liệu trên HDFS.

- `$HADOOP_HOME/etc/hadoop/yarn-site.xml` xem full cấu hình [tại đây][yarn-site]

{% highlight xml %}
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node01</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>-1</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.detect-hardware-capabilities</name>
        <value>true</value>
    </property>
</configuration>
{% endhighlight %}

---

## Chạy trên 1 node <a name="run-single-node"></a>

Format file trên Name Node

```sh
$ su hdfs
[hdfs]$ $HADOOP_HOME/bin/hdfs namenode -format
$ exit
```

Chạy các dịch vụ của Hadoop trên account root

```sh
$ $HADOOP_HOME/sbin/start-all.sh
```

Kết quả

- `http://localhost:9870/` hoặc `http://172.20.0.2:9870/`
![Name Node Web](/assets/images/blog/bigdata/2022-10-01/namenode.png)

- `http://localhost:8088/` hoặc `http://172.20.0.2:8088/`
![Yarn](/assets/images/blog/bigdata/2022-10-01/yarn.png)

---

## Thêm node mới vào cụm <a name="add-node"></a>

Để thêm một node mới vào cụm thì trên node đó cũng thực hiện đầy đủ các bước ở trên. Do sử dụng Docker nên mình sẽ tạo một image từ container đang có

```sh
$ docker commit node01 hadoop
```

Run container mới từ image vừa tạo

```sh
$ docker run -it --name node02 --hostname node02 --network hadoop hadoop
```

Trên node02 ta start service ssh và xoá thư mục data cũ đi

```sh
$ service ssh start
$ rm -rf /home/hdfs/hadoop
$ rm -rf /home/yarn/hadoop
```

Cập nhật ip, hostname của Namenode cho node02

- File `/etc/hosts`

```sh
172.20.0.3      node02
172.20.0.2      node01
```

Trên node01 chúng ta bổ sung thêm ip và hostname của node02

- File `/etc/hosts`

```sh
172.20.0.2      node01
172.20.0.3      node02
```

- File `$HADOOP_HOME/etc/hadoop/workers`

```sh
node01
node02
```

Sau đó start all các dịch vụ của hadoop trên node01

```sh
$ $HADOOP_HOME/sbin/start-all.sh
```

Kiểm tra node02 đã được add vào chưa

- `http://localhost:9870/dfshealth.html#tab-datanode`
![Datanodes](/assets/images/blog/bigdata/2022-10-01/datanodes.png)

- `http://localhost:8088/cluster/nodes`
![Yarn nodes](/assets/images/blog/bigdata/2022-10-01/yarn-nodes.png)

Làm tương tự với node03 ta sẽ được cụm 3 node
> Lưu ý do mình clone node02, node03 từ node01 ban đầu nên không cần add ssh-key của các tài khoản (do đã sử dụng chung một ssh-key). Nếu cài trên hệ thống thật thì cần copy public key từ mỗi account trên namenode và add vào authorized_keys của account tương ứng trên datanode.

---

## Hướng dẫn sử dụng cơ bản <a name="user-guide"></a>

Để start tất cả các dịch vụ trong cụm Hadoop ta cần vào master node (trong bài này là node01) sử dụng account root

```sh
$ $HADOOP_HOME/sbin/start-all.sh
```

> Master node cần có ip và hostname của tất cả các slave node trong file `/etc/hosts` và mỗi account `hdfs`, `yarn`, `mapred` của master node đều có thể ssh đến account tương ứng trên các slave node. Mỗi Slave node đều phải connect được đến Master node thông qua hostname.

Để tắt tất cả dịch vụ của cụm Hadoop

```sh
$ $HADOOP_HOME/sbin/stop-all.sh
```

---

## Kết luận <a name="conclusion"></a>

Như vậy trong bài viết này mình đã giới thiệu đầy đủ về quá trình cài Hadoop của mình, các bạn làm theo có vấn đề gì thì cố gắng tự giải quyết nha :). Hẹn gặp lại trong bài viết sau.

:memo: Hiện tại mình đang phát triển một số kênh phân tích dữ liệu blockchain hàng ngày trên nền tảng [chainslake.com](https://chainslake.com):
- [https://chainslake.com/@bitcoin](https://chainslake.com/@bitcoin)
- [https://chainslake.com/@ethereum](https://chainslake.com/@ethereum)
- [https://chainslake.com/@binance](https://chainslake.com/@binance)
- [https://chainslake.com/@aave](https://chainslake.com/@aave)
- [https://chainslake.com/@nftfi](https://chainslake.com/@nftfi)
- [https://chainslake.com/@opensea](https://chainslake.com/@opensea)
- [https://chainslake.com/@uniswap](https://chainslake.com/@uniswap)

Chainslake là nền tảng phân tích dữ liệu blockchain hoàn toàn miễn phí, do mình phát triển dựa trên những kiến thức được trình bày trong chính blog này. Rất mong sự ủng hộ từ các bạn.

[apache-hadoop]: https://hadoop.apache.org/
[hdp]: https://docs.cloudera.com/HDPDocuments/
[docker]: https://docs.docker.com/engine/install/
[download-hadoop]: https://hadoop.apache.org/releases.html
[core-site-default]: https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml
[hdfs-site]: https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
[yarn-site]: https://hadoop.apache.org/docs/current3/hadoop-yarn/hadoop-yarn-common/yarn-default.xml
[hadoop-secure]: https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SecureMode.html