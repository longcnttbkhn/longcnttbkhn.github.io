---
title: "How to install Hadoop cluster on ubuntu 20.04"
layout: post
date: 2022-10-01 10:27:10 +0700
image: /assets/images/blog/bigdata/2022-10-01/hadoop-logo.jpeg
headerImage: false
tag:
- bigdata
category: english
author: Lake Nguyen
description: In this article, I will describe in detail the process of installing and configuring Hadoop on 3 simulated nodes using Docker. I installed the latest Hadoop version (3.3.4) on Ubuntu 20.04 and Java 11 operating systems.
---

[Apache Hadoop][apache-hadoop] is an open source software project used to build big data processing systems, enabling distributed computing and scaling across clusters of up to thousands of computers with high availability and fault tolerance. Currently, Hadoop has developed into an ecosystem with many different products and services. Previously, I used [Ambari HDP][hdp] to install and manage Hadoop Ecosystem, this tool allows to centralize all configurations of Hadoop services in one place, thereby easily managing and expanding nodes when needed. However, since 2021, HDP has been closed to collect fees, all repositories require a paid account to be able to download and install. Recently, I needed to install a new Hadoop system, so I decided to manually install each component. Although it would be more complicated and time-consuming, I could control it more easily without depending on others. Partly because the new system only had 3 nodes, the workload was not too much. I will record the entire installation process in detail in a series of articles on the topic of `bigdata`. Everyone, please pay attention and read!

## Contents
1. [Target](#target)
2. [Environment setup](#environment)
3. [Download hadoop and configure](#config)
4. [Run on single node](#run-single-node)
5. [Add new node to cluster](#add-node)
6. [Basic user guide](#user-guide)
7. [Conclusion](#conclusion)


## Target <a name="target"></a>

In this article, I will install the latest Hadoop version (3.3.4 at the time of writing) on ​​3 nodes Ubuntu 20.04 and OpenJdk11. For convenience in setup and testing, I will use Docker to simulate these 3 nodes.

# Environment setup <a name="environment"></a>

First, we create a new bridge network on Docker (If you have not installed Docker, please see the installation instructions [here][docker])

```sh
$ docker network create hadoop 
```

Next is to create a container on the Ubuntu 20.04 image

```sh
$ docker run -it --name node01 -p 9870:9870 -p 8088:8088 -p 19888:19888 --hostname node01 --network hadoop ubuntu:20.04
```

> I'm using MacOS so I need to bind the port from the container to the host machine, you don't need to do this if you're using Linux or Windows.

Install the necessary packages

```sh
$ apt update
$ apt install -y wget tar ssh default-jdk 
```

Create hadoop users

```sh
$ groupadd hadoop
$ useradd -g hadoop -m -s /bin/bash hdfs
$ useradd -g hadoop -m -s /bin/bash yarn
$ useradd -g hadoop -m -s /bin/bash mapred
```

> For security reasons, Hadoop recommends that each service should run on a different user, see details [here][hadoop-secure]

Generate ssh-key on each user

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

Add hostname in file `/etc/hosts`

```sh
172.20.0.2      node01
```

> Note `172.20.0.2` is the container ip on my machine, you replace it with your machine ip.

Check if ssh is available

```sh
$ ssh <username>@node01
```

---

## Download hadoop and configure <a name="config"></a>

Go to Hadoop's download page [here][download-hadoop] to get the link to download the latest version.

```sh
$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz
$ tar -xvzf hadoop-3.3.4.tar.gz
$ mv hadoop-3.3.4 /lib/hadoop
$ mkdir /lib/hadoop/logs
$ chgrp hadoop -R /lib/hadoop
$ chmod g+w -R /lib/hadoop
```

Next, we need to configure environment variables. Here we will add environment variables to the file `/etc/bash.bashrc` so that all users on the system can use them.

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

Update environment variables

```sh
$ source /etc/bash.bashrc
```

Also need to update environment variables in the file: `$HADOOP_HOME/etc/hadoop/hadoop-env.sh`

```sh
export JAVA_HOME=/usr/lib/jvm/default-java
```

Hadoop Configuration Settings
- `$HADOOP_HOME/etc/hadoop/core-site.xml` see full configuration [here][core-site-default]

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

> `/home/${user.name}/hadoop` is the folder where I save data on HDFS, you can change to another folder if you want.
- `$HADOOP_HOME/etc/hadoop/hdfs-site.xml` see full configuration [here][hdfs-site]

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

> The `dfs.replication` configuration sets the actual number of copies stored for a data on HDFS.

- `$HADOOP_HOME/etc/hadoop/yarn-site.xml` see full configuration [here][yarn-site]

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

## Run on 1 node <a name="run-single-node"></a>

Format file on Name Node

```sh
$ su hdfs
[hdfs]$ $HADOOP_HOME/bin/hdfs namenode -format
$ exit
```

Run Hadoop services on root account

```sh
$ $HADOOP_HOME/sbin/start-all.sh
```

Result

- `http://localhost:9870/` or `http://172.20.0.2:9870/`
![Name Node Web](/assets/images/blog/bigdata/2022-10-01/namenode.png)

- `http://localhost:8088/` or `http://172.20.0.2:8088/`
![Yarn](/assets/images/blog/bigdata/2022-10-01/yarn.png)

---

## Add new node to cluster <a name="add-node"></a>

To add a new node to the cluster, perform all the steps above on that node. Because I use Docker, I will create an image from the existing container

```sh
$ docker commit node01 hadoop
```

Run new container from newly created image

```sh
$ docker run -it --name node02 --hostname node02 --network hadoop hadoop
```

On node02 we start ssh service and delete the old data folder

```sh
$ service ssh start
$ rm -rf /home/hdfs/hadoop
$ rm -rf /home/yarn/hadoop
```

Update ip, hostname of Namenode for node02

- File `/etc/hosts`

```sh
172.20.0.3      node02
172.20.0.2      node01
```

On node01 we add the ip and hostname of node02

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

Then start all hadoop services on node01

```sh
$ $HADOOP_HOME/sbin/start-all.sh
```

Check if node02 has been added

- `http://localhost:9870/dfshealth.html#tab-datanode`
![Datanodes](/assets/images/blog/bigdata/2022-10-01/datanodes.png)

- `http://localhost:8088/cluster/nodes`
![Yarn nodes](/assets/images/blog/bigdata/2022-10-01/yarn-nodes.png)

Do the same with node03 and you will get a cluster of 3 nodes
> Note that because I cloned node02, node03 from the original node01, there is no need to add the ssh-key of the accounts (because they already use the same ssh-key). If installed on a real system, you need to copy the public key from each account on the namenode and add it to the authorized_keys of the corresponding account on the datanode.

---

## Basic User Guide <a name="user-guide"></a>

To start all services in the Hadoop cluster, we need to go to the master node (in this article, node01) using the root account

```sh
$ $HADOOP_HOME/sbin/start-all.sh
```

> The master node needs to have the ip and hostname of all slave nodes in the `/etc/hosts` file and each `hdfs`, `yarn`, `mapred` account of the master node can ssh to the corresponding account on the slave nodes. Each Slave node must be able to connect to the Master node via hostname.

To turn off all services of the Hadoop cluster

```sh
$ $HADOOP_HOME/sbin/stop-all.sh
```

---

## Conclusion <a name="conclusion"></a>

So in this article I have fully introduced my Hadoop installation process, if you have any problems following, please try to solve them yourself :). See you in the next article.

[apache-hadoop]: https://hadoop.apache.org/
[hdp]: https://docs.cloudera.com/HDPDocuments/
[docker]: https://docs.docker.com/engine/install/
[download-hadoop]: https://hadoop.apache.org/releases.html
[core-site-default]: https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml
[hdfs-site]: https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
[yarn-site]: https://hadoop.apache.org/docs/current3/hadoop-yarn/hadoop-yarn-common/yarn-default.xml
[hadoop-secure]: https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SecureMode.html