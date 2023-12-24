---
title: 'Hadoop 大数据平台搭建过程'
date: '2023-12-20 09:37:11'
tags: ['大数据','Hadoop']
---

# 集群规划

| 节点 | hadoop101     | hadoop102         | hadoop103       |
| ---- | ------------- | ----------------- | --------------- |
| IP   | 192.168.1.101 | 192.168.1.102     | 192.168.1.103   |
| 配置 | 8C 16G        | 4C 8G             | 4C 8G           |
|      | NameNode      |                   |                 |
|      |               | SecondaryNameNode |                 |
|      | DataNode      | DataNode          | DataNode        |
|      |               |                   | ResourceManager |
|      | NodeManager   | NodeManager       | NodeManager     |



# 安装系统

## 下载系统镜像

[Anolis OS 8.8](https://mirrors.openanolis.cn/anolis/8.8/isos/GA/x86_64/AnolisOS-8.8-x86_64-minimal.iso)

## 创建虚拟机

创建虚拟机 hadoop101

![image-20231224113345710](https://oss.lzhui.top/blog/image-20231224113345710.png)

## 系统通用配置

1. 安装必要的软件

    ```bash
    dnf update
    dnf install -y tar vim wget
    ```

2. 配置 hosts

    ```bash
    vim /etc/hosts
    ```

    ```bash
    192.168.1.101 hadoop101
    192.168.1.102 hadoop102
    192.168.1.103 hadoop103
    ```

3. 配置主机名

    ```bash
    vim /etc/hostname
    ```

    ```bash
    hadoop101
    ```

4. 配置静态地址

    ```bash
    vim /etc/sysconfig/network-scripts/ifcfg-ens33
    ```

    ```bash
    ......
    IPADDR=192.168.1.101
    PREFIX=24
    GATEWAY=192.168.1.1
    DNS1=192.168.1.1
    ```
    
5. 关闭防火墙

    ```bash
    systemctl stop firewalld
    systemctl disable firewalld
    setenforce 0
    iptables -F
    ```

## 创建 hadoop 用户

1. 创建用户家目录

    ```bash
    mkdir /user
    ```

2. 创建用户

    ```bash
    useradd -d /user/hadoop -m hadoop
    ```

3. 设置密码

    ```bash
    passwd hadoop
    ```

4. 添加 sudo 权限

    ```bash
    visudo
    hadoop  ALL=(ALL)       NOPASSWD: ALL
    ```

## 克隆虚拟机

1. 创建 hadoop101 的快照，克隆虚拟机 hadoop102、hadoop103，并修改为 4C 8G

   ![image-20231224113519873](https://oss.lzhui.top/blog/image-20231224113519873.png)

2. 修改 hadoop102、hadoop103 的 IP 和主机名

3. 登录三台服务器，配置 SSH 免密登录

   ```bash
   # hadoop 用户
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   ssh-copy-id hadoop@hadoop101
   ssh-copy-id hadoop@hadoop102
   ssh-copy-id hadoop@hadoop103
   ```

# 安装集群

使用 MobaXterm 的 Multi-execution mode 功能同时操作三台服务器

![image-20231224182417167](https://oss.lzhui.top/blog/image-20231224182417167.png)

## 安装 Java 11

1. 下载 [Java 11](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html)

2. 解压并配置环境变量

    ```bash
    sudo tar xvf jdk-11.0.20_linux-x64_bin.tar.gz -C /usr/local/
    sudo vim /etc/profile.d/server.sh
    
    # JAVA 17
    export JAVA_HOME=/usr/local/jdk-11.0.20
    export PATH=$PATH:$JAVA_HOME/bin
    
    source /etc/profile
    java -version
    ```

## 安装 Hadoop 3.3.6

1. 下载 [Hadoop 3.3.6](https://mirrors.bfsu.edu.cn/apache/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz)
2. 解压并配置环境变量

    ```bash
    sudo tar xvf hadoop-3.3.6.tar.gz -C /usr/local/

    sudo vim /etc/profile.d/server.sh
    # HADOOP 3.3.6
    export HADOOP_HOME=/usr/local/hadoop-3.3.6
    export PATH=$PATH:$HADOOP_HOME/bin
    export PATH=$PATH:$HADOOP_HOME/sbin

    source /etc/profile
    hadoop version
    ```

3. 修改配置文件

    `etc/hadoop/hadoop-env.sh`

    ```bash
    export JAVA_HOME=/usr/local/jdk-11.0.20
    ```

    `etc/hadoop/core-site.xml`

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop101:9000</value>
        </property>
    </configuration>
    ```

    `etc/hadoop/hdfs-site.xml`

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
    </configuration>
    ```

    `etc/hadoop/mapred-site.xml`

    ```xml
    <?xml version="1.0"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
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

    `etc/hadoop/yarn-site.xml`

    ```xml
    <?xml version="1.0"?>
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

4. 初始化

    ```bash
    hdfs namenode -format
    ```

5. 启动

    ```bash
    start-dfs.sh
    hdfs dfs -mkdir -p /user/hadoop
    ```

6. 访问

- [http://192.168.1.101:9870](http://192.168.194.101:9870)

- [http://192.168.1.101:8088](http://192.168.194.101:9870)