---
title: 'Hadoop 大数据平台搭建过程（单机）'
date: '2023-12-29'
tags: ['大数据','Hadoop','Hive','Spark']
---

## 系统配置

### 软件

```bash
dnf install -y tar vim wget
```

### 网络

```bash
vim /etc/hosts
192.168.1.10 data
```

### 防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
iptables -F
```

### 用户

```bash
mkdir /user
useradd -d /user/hadoop -m hadoop
passwd hadoop

visudo
hadoop  ALL=(ALL)       NOPASSWD: ALL

su hadoop
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

## 组件安装

### Java

https://www.oracle.com/cn/java/technologies/javase/javase8u211-later-archive-downloads.html

```bash
tar xvf jdk-8u381-linux-x64.tar.gz
sudo mv jdk1.8.0_381 /opt/java

sudo vim /etc/profile.d/server.sh
# JAVA
export JAVA_HOME=/opt/java
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```

### MySQL

```bash
sudo dnf install -y mysql-server
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo mysqladmin -u root -p password

mysql -uroot -p123456
create database metastore;
exit
```

### Hadoop

```bash
tar xvf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /opt/hadoop

sudo vim /etc/profile.d/server.sh
# HADOOP
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

source /etc/profile
```

$HADOOP_HOME/etc/hadoop/workers

```bash
data
```

$HADOOP_HOME/etc/hadoop/hadoop-env.sh

```bash
export JAVA_HOME=/opt/java
```

$HADOOP_HOME/etc/hadoop/core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://data:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop/tmp</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hadoop.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hadoop.groups</name>
        <value>*</value>
    </property>
</configuration>
```

$HADOOP_HOME/etc/hadoop/hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

$HADOOP_HOME/etc/hadoop/mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

$HADOOP_HOME/etc/hadoop/yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

```bash
hdfs namenode -format
start-dfs.sh
start-yarn.sh
hdfs dfs -mkdir -p /user/{hadoop,hive/warehouse,tez}
hdfs dfs -chown -R hadoop:hadoop /user
```

### Hive

```bash
tar xvf apache-hive-3.1.3-bin.tar.gz
sudo mv apache-hive-3.1.3-bin /opt/hive

sudo vim /etc/profile.d/server.sh
# HIVE
export HIVE_HOME=/opt/hive
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile

beeline -u jdbc:hive2://data:10000 -n hadoop
```

$HIVE_HOME/conf/hive-site.xml

```xml
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://127.0.0.1:3306/metastore?useSSL=false</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://data:9083</value>
    </property>
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>data</value>
    </property>
    <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
    </property>
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.execution.engine</name>
        <value>tez</value>
    </property>
</configuration>
```

```bash
wget -O $HIVE_HOME/lib/mysql-connector-java-5.1.49.jar https://oss.lzhui.top/common/mysql-connector-java-5.1.49.jar
schematool -initSchema -dbType mysql -verbose
mkdir -p $HIVE_HOME/log
nohup hive --service metastore > $HIVE_HOME/log/metastore.log 2>&1 &
nohup hive --service hiveserver2 > $HIVE_HOME/log/hiveserver2.log 2>&1 &
```

### Tez

```bash
tar xvf apache-tez-0.10.2-bin.tar.gz
sudo mv apache-tez-0.10.2-bin /opt/tez
cd /opt/tez/share
hdfs dfs -put tez.tar.gz /user/tez
```

$HADOOP_HOME/etc/hadoop/tez-site.xml

```xml
<configuration>
  <property>
    <name>tez.lib.uris</name>
    <value>hdfs://data:9000/user/tez/tez.tar.gz</value>
  </property>
</configuration>
```

```bash
sudo vim /etc/profile.d/server.sh
# TEZ
export TEZ_CONF_DIR=$HADOOP_CONF_DIR
export TEZ_JARS=/opt/tez/*:/opt/tez/lib/*
export HADOOP_CLASSPATH=$TEZ_CONF_DIR:$TEZ_JARS:$HADOOP_CLASSPATH
```



### HBase

### Spark

```bash
tar xvf spark-3.5.0-bin-hadoop3-scala2.13.tgz
sudo mv spark-3.5.0-bin-hadoop3-scala2.13 /opt/spark

sudo vim /etc/profile.d/server.sh
# SPARK
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin

source /etc/profile
```

### Flink