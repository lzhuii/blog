---
title: 'Linux 系统中安装 Gitea'
date: '2023-12-06 08:04:10'
tags: ['Linux','Git']
---

1. 创建数据库

   ```bash
   mysql -u root -p
   ```

   ```sql
   CREATE USER 'gitea' IDENTIFIED BY '密码';
   CREATE DATABASE gitea CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
   GRANT ALL PRIVILEGES ON gitea.* TO 'gitea';
   FLUSH PRIVILEGES;
   ```

2. 创建工作路径，下载 Gitea，安装 Git，创建 git 用户

   ```bash
   sudo mkdir -p /usr/local/gitea/{bin,conf,custom,data,log}
   sudo wget -O /usr/local/gitea/bin/gitea https://dl.gitea.com/gitea/1.21.0/gitea-1.21.0-linux-amd64
   sudo chmod +x /usr/local/gitea/bin/gitea
   sudo dnf install -y git
   sudo useradd git
   sudo chown -R git:git /usr/local/gitea
   sudo chmod -R 750 /usr/local/gitea
   ```

3. 创建 Gitea 服务

   ```bash
   sudo vim /etc/systemd/system/gitea.service
   ```

   ```bash
   [Unit]
   Description=Gitea
   After=syslog.target
   After=network.target
   
   Wants=mysqld.service
   After=mysqld.service
   
   [Service]
   User=git
   Group=git
   Type=simple
   WorkingDirectory=/usr/local/gitea
   Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/usr/local/gitea
   ExecStart=/usr/local/gitea/bin/gitea web --config /usr/local/gitea/conf/app.ini
   Restart=always
   RestartSec=1s
   
   [Install]
   WantedBy=multi-user.target
   ```

4. 配置 Nginx 代理

   ```bash
   sudo vim /usr/local/nginx/conf/conf.d/gitea.conf
   ```

   ```nginx
   server {
       listen 443 ssl http2;
       listen [::]:443 ssl http2;
       
       server_name git.lzhui.top;
   
       ssl_certificate cert/lzhui.top.pem;
       ssl_certificate_key cert/lzhui.top.key;
   
       ssl_session_cache shared:SSL:1m;
       ssl_session_timeout 5m;
       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
       ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
       ssl_prefer_server_ciphers on;
   
       location / {
           proxy_pass http://127.0.0.1:3000/;
           
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

5. 启动 Gitea

   ```bash
   sudo systemctl restart nginx
   sudo systemctl start gitea
   sudo systemctl enable gitea
   ```

6. 下载，安装 act_runner

   ```bash
   mkdir -p /opt/act_runner
   wget -O /opt/act_runner/act_runner https://dl.gitea.com/act_runner/0.2.6/act_runner-0.2.6-linux-amd64
   chmod +x /opt/act_runner/act_runner
   ```

7. 配置 act_runner

   ```bash
   # /opt/gitea/conf/app.ini
   [actions]
   ENABLED=true
   
   cd /opt/act_runner
   ./act_runner generate-config > config.yaml
   ./act_runner -c config.yaml register
   # gitea地址 https://git.lzhui.top
   # token
   # runner名称 默认
   # 标签 linux
   ```

8. 启动 act_runner

   ```bash
   chown -R git:git /opt/act_runner
   su git
   nohup /opt/act_runner/act_runner daemon --config /opt/act_runner/config.yaml > /opt/act_runner/act_runner.log 2>&1 &
   ```

   