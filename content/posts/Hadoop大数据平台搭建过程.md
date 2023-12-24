---
title: 'Hadoop 大数据平台搭建过程'
date: '2023-12-20 09:37:11'
tags: ['大数据','Hadoop']
---

## 集群规划

| 节点      | IP            | 配置   | 组件                                    |
| --------- | ------------- | ------ | --------------------------------------- |
| hadoop101 | 192.168.1.101 | 8C 16G | MySQL,NameNode,DataNode,ResourceManager |
| hadoop102 | 192.168.1.102 | 4C 8G  | DataNode                                |
| hadoop103 | 192.168.1.103 | 4C 8G  | DataNode                                |

# 一、安装系统

## 1.1 创建虚拟机

1. 下载操作系统镜像 [Anolis OS 8.8](https://mirrors.openanolis.cn/anolis/8.8/isos/GA/x86_64/AnolisOS-8.8-x86_64-minimal.iso)

2. 创建虚拟机 hadoop101，克隆虚拟机 hadoop102,hadoop103

   ![image-20231224104359087](https://oss.lzhui.top:443/blog/image-20231224104359087.png)

   ![image-20231224104417014](https://oss.lzhui.top:443/blog/image-20231224104417014.png)

   ![image-20231224104532251](https://oss.lzhui.top:443/blog/image-20231224104532251.png)

   ![image-20231224104541013](https://oss.lzhui.top:443/blog/image-20231224104541013.png)

   ![image-20231224104554978](https://oss.lzhui.top:443/blog/image-20231224104554978.png)

   ![image-20231224104632139](https://oss.lzhui.top:443/blog/image-20231224104632139.png)

   

3. 安装必要的软件

   ```bash
   dnf install -y tar vim
   ```

## 1.2 配置 hosts

```bash
vim /etc/hosts

192.168.194.101 hadoop101
```

## 1.3 配置主机名

```bash
vim /etc/hostname

hadoop101
```

## 1.4 配置静态地址

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=ens33
UUID=19ee12a2-8b4c-4d9f-a687-ac9017395936
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.194.101
PREFIX=24
GATEWAY=192.168.194.147
DNS1=192.168.194.147
```

## 1.5 关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
iptables -F
```

## 1.6 创建 hadoop 用户

```bash
# 创建用户家目录
mkdir /user
# 创建用户
useradd -d /user/hadoop -m hadoop
# 设置密码
passwd hadoop
# 123456
# 123456

# 添加 sudo 权限
visudo

hadoop  ALL=(ALL)       NOPASSWD: ALL
```

## 1.7 配置 SSH 免密登录

```bash
# 切换到 hadoop 用户
su hadoop
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```


# 二、安装 Java 11

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


# 三、安装 Hadoop 3.3.6

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

   - [http://192.168.194.101:9870](http://192.168.194.101:9870)

   - [http://192.168.194.101:8088](http://192.168.194.101:9870)