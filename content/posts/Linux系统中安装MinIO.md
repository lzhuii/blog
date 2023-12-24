---
title: 'Linux 系统中安装 MinIO'
date: '2023-12-24 09:36:10'
tags: ['Linux',MinIO']
---

1. 创建安装目录

   ```bash
   sudo mkdir -p /usr/local/minio/{bin,conf,data}
   ```

2. 创建 minio 用户

   ```bash
   sudo useradd minio
   sudo chown -R minio:minio /usr/local/minio
   su minio
   ```

3. 下载 MinIO

   ```bash
   wget -O /usr/local/minio/bin/minio https://dl.min.io/server/minio/release/linux-amd64/minio
   wget -O /usr/local/minio/bin/mc https://dl.min.io/client/mc/release/linux-amd64/mc
   ```

4. 创建 MinIO 配置文件

   ```bash
   vim /usr/local/minio/conf/minio.conf
   ```

   ```ini
   MINIO_ROOT_USER=用户
   MINIO_ROOT_PASSWORD=密码
   MINIO_VOLUMES="/usr/local/minio/data"
   MINIO_SERVER_URL="https://oss.lzhui.top"
   MINIO_BROWSER_REDIRECT_URL="https://oss.lzhui.top/minio/ui"
   MINIO_OPTS="--address 0.0.0.0:9000 --console-address 0.0.0.0:9001"
   ```

5. 创建 MinIO 服务

   ```bash
   sudo vim /etc/systemd/system/minio.service
   ```

   ```bash
   [Unit]
   Description=MinIO
   Documentation=https://min.io/docs/minio/linux/index.html
   Wants=network-online.target
   After=network-online.target
   AssertFileIsExecutable=/usr/local/minio/bin/minio
   
   [Service]
   WorkingDirectory=/usr/local
   User=minio
   Group=minio
   ProtectProc=invisible
   EnvironmentFile=-/user/local/minio/conf/minio.conf
   ExecStart=/usr/local/minio/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
   Restart=always
   LimitNOFILE=65536
   TasksMax=infinity
   TimeoutStopSec=infinity
   SendSIGKILL=no
   
   [Install]
   WantedBy=multi-user.target
   ```

6. 配置 Nginx 代理

   ```bash
   sudo vim /usr/local/nginx/conf/conf.d/minio.conf
   ```

   ```bash
   server {
       listen 443 ssl;
       listen [::]:443 ssl;
   
       server_name oss.lzhui.top;
   
       ssl_certificate cert/lzhui.top.pem;
       ssl_certificate_key cert/lzhui.top.key;
   
       ssl_session_cache shared:SSL:1m;
       ssl_session_timeout 5m;
       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
       ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
       ssl_prefer_server_ciphers on;
   
      # Allow special characters in headers
      ignore_invalid_headers off;
      # Allow any size file to be uploaded.
      # Set to a value such as 1000m; to restrict file size to a specific value
      client_max_body_size 0;
      # Disable buffering
      proxy_buffering off;
      proxy_request_buffering off;
   
      location / {
         proxy_set_header Host $http_host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-Forwarded-Proto $scheme;
   
         proxy_connect_timeout 300;
         # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
         proxy_http_version 1.1;
         proxy_set_header Connection "";
         chunked_transfer_encoding off;
   
         proxy_pass http://127.0.0.1:9000/; # This uses the upstream directive definition to load balance
      }
   
      location /minio/ui/ {
         rewrite ^/minio/ui/(.*) /$1 break;
         proxy_set_header Host $http_host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-Forwarded-Proto $scheme;
         proxy_set_header X-NginX-Proxy true;
   
         # This is necessary to pass the correct IP to be hashed
         real_ip_header X-Real-IP;
   
         proxy_connect_timeout 300;
   
         # To support websockets in MinIO versions released after January 2023
         proxy_http_version 1.1;
         proxy_set_header Upgrade $http_upgrade;
         proxy_set_header Connection "upgrade";
         # Some environments may encounter CORS errors (Kubernetes + Nginx Ingress)
         # Uncomment the following line to set the Origin request to an empty string
         # proxy_set_header Origin '';
   
         chunked_transfer_encoding off;
   
         proxy_pass http://127.0.0.1:9001/; # This uses the upstream directive definition to load balance
      }
   }
   ```

7. 启动

   ```bash
   sudo systemctl start minio
   sudo systemctl enable minio
   ```

   
