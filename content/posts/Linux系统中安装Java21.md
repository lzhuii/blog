---
title: 'Linux 系统中安装 Java 21'
date: '2023-11-30 00:48:00'
tags: ['Linux','Java']
---

1. 下载  Java 17

   ```bash
   wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
   ```

2. 解压  Java 17

   ```bash
   sudo tar xvf jdk-21_linux-x64_bin.tar.gz -C /usr/local
   ```

3. 配置环境变量

   ```bash
   sudo vim /etc/profile.d/ecs.sh
   ```

   ```bash
   # JAVA 21
   export JAVA_HOME=/usr/local/jdk-21.0.1
   export PATH=$PATH:$JAVA_HOME/bin
   ```

4. 刷新环境变量

   ```bash
   source /etc/profile
   ```

5. 验证是否安装成功

   ```bash
   java -version
   ```
