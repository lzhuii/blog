1. 安装 MySQL

   ```bash
   dnf install -y mysql-server
   ```

2. 启动 MySQL

   ```bash
   systemctl start mysqld
   ```

3. 修改默认端口

   ```bash
   # /etc/my.cnf.d/mysql-server.cnf
   [mysqld]
   port=端口
   ```

4. 重启 MySQL

   ```bash
   systemctl restart mysqld
   ```

5. 修改 root 密码

   ```bash
   mysqladmin -u root -p password
   #旧密码(默认是空)
   #新密码
   #确认密码
   ```

6. 创建远程登陆用户

   ```bash
   CREATE USER '用户'@'%' IDENTIFIED BY '密码';
   ```