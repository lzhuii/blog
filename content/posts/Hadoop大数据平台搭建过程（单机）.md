---
title: 'Hadoop 大数据平台搭建过程（单机）'
date: '2023-12-29'
tags: ['大数据','Hadoop','Hive','Spark']. a阿坝州
---

## 系统配置

```bash
# 软件
dnf install -y tar vim wget

# 网络
vim /etc/hosts
192.168.1.10 data

# 防火墙
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
iptables -F

# 用户
mkdir /user
useradd -d /user/hadoop -m hadoop
passwd hadoop

# 权限
visudo
hadoop  ALL=(ALL)       NOPASSWD: ALL

su hadoop
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
ssh-copy-id data

```

# 组件安装

 [Java 8](https://www.oracle.com/cn/java/technologies/javase/javase8u211-later-archive-downloads.html)

```bash
tar xvf jdk-8u381-linux-x64.tar.gz
tar xvf hadoop-3.3.6.tar.gz
tar xvf apache-hive-3.1.3-bin.tar.gz
tar xvf apache-tez-0.10.2-bin.tar.gz
tar xvf spark-3.5.0-bin-hadoop3-scala2.13.tgz

sudo mv jdk1.8.0_381 /opt/java
sudo mv hadoop-3.3.6 /opt/hadoop
sudo mv apache-hive-3.1.3-bin /opt/hive
sudo mv apache-tez-0.10.2-bin /opt/tez
sudo mv spark-3.5.0-bin-hadoop3-scala2.13 /opt/spark

sudo vim /etc/profile.d/server.sh
# JAVA
export JAVA_HOME=/opt/java
export PATH=$PATH:$JAVA_HOME/bin

# HADOOP
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

# HIVE
export HIVE_HOME=/opt/hive
export PATH=$PATH:$HIVE_HOME/bin

# TEZ
export TEZ_HOME=/opt/tez
export PATH=$PATH:$TEZ_HOME/bin

# SPARK
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin

source /etc/profile
java -version
hadoop version
hive --version

sudo dnf install -y mysql-server
sudo systemctl start mysqld
sudo mysqladmin -u root -p password
mysql -uroot -p123456
create database metastore;
exit
```

**$HADOOP_HOME/etc/hadoop/workers**

```bash
data
```

**$HADOOP_HOME/etc/hadoop/core-site.xml**

**$HADOOP_HOME/etc/hadoop/hdfs-site.xml**

**$HADOOP_HOME/etc/hadoop/mapred-site.xml**

**$HADOOP_HOME/etc/hadoop/yarn-site.xml**

**$HIVE_HOME/conf/hive-site.xml**