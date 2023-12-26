---
title: 'Hadoop 大数据平台搭建过程'
date: '2023-12-20'
tags: ['大数据','Hadoop']
---

# 集群规划

| 节点 | hadoop101                                      | hadoop102                                        | hadoop103                                      |
| ---- | ---------------------------------------------- | ------------------------------------------------ | ---------------------------------------------- |
| IP   | 192.168.1.101                                  | 192.168.1.102                                    | 192.168.1.103                                  |
| 配置 | 8C 16G                                         | 4C 8G                                            | 4C 8G                                          |
| 组件 | NameNode<br />NodeManager<br />MySQL<br />Hive | SecondaryNameNode<br />DataNode<br />NodeManager | DataNode<br />ResourceManager<br />NodeManager |

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

## 安装 Java 8

1. 下载 [Java 8](https://www.oracle.com/cn/java/technologies/javase/javase8u211-later-archive-downloads.html)

2. 解压并配置环境变量

    ```bash
    sudo tar xvf jdk-8u381-linux-x64.tar.gz -C /usr/local/
    sudo vim /etc/profile.d/server.sh
    
    # JAVA 8
    export JAVA_HOME=/usr/local/jdk1.8.0_381
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
    export JAVA_HOME=/usr/local/jdk1.8.0_381
    ```

    `etc/hadoop/core-site.xml`

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop101:8020</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/usr/local/hadoop-3.3.6/data</value>
        </property>
        <property>
            <name>hadoop.http.staticuser.user</name>
            <value>hadoop</value>
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

    `etc/hadoop/hdfs-site.xml`

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>dfs.namenode.http-address</name>
            <value>hadoop101:9870</value>
        </property>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>hadoop102:9868</value>
        </property>
    </configuration>
    ```

    `etc/hadoop/mapred-site.xml`

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    ```

    `etc/hadoop/yarn-site.xml`

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>hadoop103</value>
        </property>
        <property>
            <name>yarn.nodemanager.env-whitelist</name>
            <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
        </property>
    </configuration>
    ```

    `etc/hadoop/workers`

    ```bash
    hadoop101
    hadoop102
    hadoop103
    ```

4. 初始化

    ```bash
    # hadoop101
    hdfs namenode -format
    ```

5. 启动 HDFS

    ```bash
    # hadoop101
    start-dfs.sh
    ```

6. 启动 YARN

    ```bash
    # hadoop103
    start-yarn.sh
    ```

7. 检查 HDFS 和 YARN 的进程是否正确启动

    ![image-20231224195156615](https://oss.lzhui.top/blog/image-20231224195156615.png)

8. 访问

- [http://hadoop101:9870](http://hadoop101:9870)

- [http://hadoop103:8088](http://hadoop103:8088)

## 安装 MySQL 8.0

1. 安装 mysql-server

   ```bash
   sudo dnf install -y mysql-server
   ```

2. 启动 mysqld

   ```bash
   sudo systemctl start mysqld
   ```

3. 修改 root 密码

   ```bash
   sudo mysqladmin -u root -p password
   # 默认为空
   # 123456
   # 123456
   ```

4. 创建数据库并开启远程访问

   ```bash
   mysql -uroot -p123456
   use mysql;
   update user set host = '%' where user = 'root';
   flush privileges;
   create database metastore;
   exit
   ```

   

## 安装 Hive 3.1.3

1. 下载 [Hive 3.1.3](https://mirrors.bfsu.edu.cn/apache/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz)

2. 解压并配置环境变量

   ```bash
   sudo tar xvf apache-hive-3.1.3-bin.tar.gz -C /usr/local
   sudo mv /usr/local/apache-hive-3.1.3-bin /usr/local/hive-3.1.3
   sudo chown -R hadoop:hadoop /usr/local/hive-3.1.3
   sudo vim /etc/profile.d/server.sh
   
   # HIVE 3.1.3
   export HIVE_HOME=/usr/local/hive-3.1.3
   export PATH=$PATH:$HIVE_HOME/bin
   
   source /etc/profile
   hive --version
   ```

3. 创建 hive 配置文件 `conf/hive-site.xml`

   ```xml
   <?xml version="1.0"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   
   <configuration>
     <!-- jdbc 连接的 URL -->
     <property>
       <name>javax.jdo.option.ConnectionURL</name>
       <value>jdbc:mysql://hadoop101:3306/metastore?useSSL=false</value>
     </property>
     <!-- jdbc 连接的 Driver-->
     <property>
       <name>javax.jdo.option.ConnectionDriverName</name>
       <value>com.mysql.jdbc.Driver</value>
     </property>
     <!-- jdbc 连接的 username-->
     <property>
       <name>javax.jdo.option.ConnectionUserName</name>
       <value>root</value>
     </property>
     <!-- jdbc 连接的 password -->
     <property>
       <name>javax.jdo.option.ConnectionPassword</name>
       <value>123456</value>
     </property>
     <!-- Hive 元数据存储版本的验证 -->
     <property>
       <name>hive.metastore.schema.verification</name>
       <value>false</value>
     </property>
     <!-- 元数据存储授权 -->
     <property>
       <name>hive.metastore.event.db.notification.api.auth</name>
       <value>false</value>
     </property>
     <!-- Hive 默认在 HDFS 的工作目录 -->
     <property>
       <name>hive.metastore.warehouse.dir</name>
       <value>/user/hive/warehouse</value>
     </property>
     <!-- 指定存储元数据要连接的地址 -->
     <property>
       <name>hive.metastore.uris</name>
       <value>thrift://hadoop101:9083</value>
     </property>
     <!-- 指定 hiveserver2 连接的 host -->
     <property>
       <name>hive.server2.thrift.bind.host</name>
       <value>hadoop101</value>
     </property>
     <!-- 指定 hiveserver2 连接的端口号 -->
     <property>
       <name>hive.server2.thrift.port</name>
       <value>10000</value>
     </property>
     <!--打印当前库和表头-->
     <property>
       <name>hive.cli.print.header</name>
       <value>true</value>
     </property>
     <property>
       <name>hive.cli.print.current.db</name>
       <value>true</value>
     </property>
   </configuration>
   ```

4. 安装 mysql jdbc 驱动

   ```bash
   sudo wget -O $HIVE_HOME/lib/mysql-connector-java-5.1.49.jar https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.49/mysql-connector-java-5.1.49.jar
   ```

5. 初始化 hive 元数据库

   ```bash
   schematool -initSchema -dbType mysql -verbose
   ```

6. 启动 hive

   ```bash
   mkdir -p $HIVE_HOME/log
   nohup hive --service metastore > $HIVE_HOME/log/metastore.log 2>&1 &
   nohup hive --service hiveserver2 > $HIVE_HOME/log/hiveserver2.log 2>&1 & 
   ```

   