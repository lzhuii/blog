---
title: Linux 系统中安装 Maven
date: 2023-11-30
tags: ['Linux','Maven']
---

1. 下载 Maven

   ```bash
   wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
   ```

2. 解压 Maven

   ```bash
   sudo tar xvf apache-maven-3.9.6-bin.tar.gz -C /usr/local
   ```

3. 配置环境变量

   ```bash
   sudo vim /etc/profile.d/ecs.sh
   ```

   ```bash
   # MAVEN
   export MAVEN_HOME=/usr/local/apache-maven-3.9.6
   export PATH=$PATH:$MAVEN_HOME/bin
   ```

4. 刷新环境变量

   ```bash
   source /etc/profile
   ```

5. 验证

   ```bash
   mvn -v
   ```

