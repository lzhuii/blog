---
title: 'Linux 系统中安装 MySQL 8.0'
date: '2023-11-30 00:13:00'
tags: ['Linux','MySQL']
---

1. 安装 MySQL

   ```bash
   sudo dnf install -y mysql-server
   ```

2. 修改默认端口

   ```bash
   sudo vim /etc/my.cnf.d/mysql-server.cnf
   ```

   ```bash
   [mysqld]
   port=端口
   ```

3. 启动 MySQL

   ```bash
   sudo systemctl start mysqld
   ```

4. 修改 root 密码

   ```bash
   sudo mysqladmin -u root -p password
   #旧密码(默认是空)
   #新密码
   #确认密码
   ```

5. 创建远程登陆用户

   ```bash
   CREATE USER '用户'@'%' IDENTIFIED BY '密码';
   ```