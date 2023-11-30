---
title: 'Linux 系统中安装 Maven'
date: '2023-11-30 09:01:00'
---

1. 下载 Maven

   ```bash
   wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
   ```

2. 解压 Maven

   ```bash
   tar xvf apache-maven-3.9.5-bin.tar.gz -C /usr/locacl
   ```

3. 配置环境变量 `/etc/profile`

   ```bash
   # MAVEN
   export MAVEN_HOME=/usr/local/apache-maven-3.9.5
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

