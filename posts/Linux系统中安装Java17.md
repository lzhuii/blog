---
title: 'Linux 系统中安装 Java 17'
date: '2023-11-30 00:48:00'
---

1. 下载  Java 17

   ```bash
   wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
   ```

2. 解压  Java 17

   ```bash
   tar xvf jdk-17_linux-x64_bin.tar.gz -C /usr/local
   ```

3. 配置环境变量 `/etc/profile`

   ```bash
   # JAVA 17
   export JAVA_HOME=/usr/local/jdk-17.0.9
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

![](https://oss.lzhui.top:443/note/202311300052367.png)

