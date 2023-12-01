---
title: 'Linux 系统中安装 Nginx'
date: '2023-11-30 12:04:00'
tags: ['Linux','Nginx']
---

1. 安装依赖

   ```bash
   dnf install -y zlib-devel pcre-devel openssl-devel
   ```

2. 下载 Nginx 源码

   ```bash
   wget https://nginx.org/download/nginx-1.24.0.tar.gz
   ```

3. 解压 Nginx 源码

   ```bash
   tar xvf nginx-1.24.0.tar.gz
   ```

4. 配置

   ```bash
   cd nginx-1.24.0
   ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module
   ```

5. 编译安装

   ```bash
   make -j 4
   make install
   ```

6. 安装 SSL 证书

   ```bash
   cd /usr/local/nginx/conf
   mkdir cert
   # sftp上传证书
   ```

7. 修改 Nginx 配置文件 `/usr/local/nginx/conf/nginx.conf`

   ```nginx
   user nginx;
   
   worker_processes 1;
   
   events {
      worker_connections 1024;
   }
   
   http {
      include mime.types;
      default_type application/octet-stream;
      sendfile on;
      keepalive_timeout 65;
      gzip on;
      
      server {
          listen 80;
          listen [::]:80;
          server_name 域名;
   
          return 301 https://$host$request_uri;
      }
   
      server {
         listen 443 ssl http2;
         listen [::]:443 ssl http2;
   
         server_name 域名;
   
         ssl_certificate cert/域名.pem;
         ssl_certificate_key cert/域名.key;
   
         ssl_session_cache shared:SSL:1m;
         ssl_session_timeout 5m;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
         ssl_prefer_server_ciphers on;
   
         location / {
            root html;
            index index.html index.htm;
         }
      }
   
      include conf.d/*.conf;
   }
   ```

8. 创建 nginx 用户

   ```bash
   useradd nginx
   chown -R nginx:nginx /usr/local/nginx 
   setcap 'cap_net_bind_service=+ep' /usr/local/nginx/sbin/nginx
   ```

9. 创建 nginx 服务 `/etc/systemd/system/nginx.service`

   ```bash
   [Unit]
   Description=Nginx
   After=network.target
   
   [Service]
   User=nginx
   Group=nginx
   Type=forking
   ExecStart=/usr/local/nginx/sbin/nginx
   ExecReload=/usr/local/nginx/sbin/nginx -s reload
   ExecStop=/usr/local/nginx/sbin/nginx -s stop
   KillSignal=SIGQUIT
   TimeoutStopSec=5
   KillMode=process
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   ```

10. 启动 Nginx

    ```bash
    systemctl start nginx
    systemctl enable nginx
    ```

    