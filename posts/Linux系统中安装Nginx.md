---
title: 'Linux 系统中安装 Nginx'
date: '2023-11-30 12:04:00'
tags: ['Linux','Nginx']
---

1. 安装依赖

   ```bash
   sudo dnf install -y zlib-devel pcre-devel openssl-devel
   ```

2. 下载 Nginx 源码

   ```bash
   wget https://nginx.org/download/nginx-1.24.0.tar.gz
   ```

3. 解压 Nginx 源码

   ```bash
   sudo tar xvf nginx-1.24.0.tar.gz -C /usr/local/src
   ```

4. 编译安装

   ```bash
   cd /usr/local/src/nginx-1.24.0
   sudo ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module
   sudo make -j 4
   sudo make install
   ```

5. 创建 nginx 用户

   ```bash
   sudo useradd nginx
   ```

7. 安装 SSL 证书

   ```bash
   sudo mkdir /usr/local/nginx/conf/cert
   # sftp上传证书
   ```

8. 修改 Nginx 配置文件

   ```bash
   sudo vim /usr/local/nginx/conf/nginx.conf
   ```

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
          server_name lzhui.top;
   
          return 301 https://$host$request_uri;
      }
   
      server {
         listen 443 ssl http2;
         listen [::]:443 ssl http2;
   
         server_name lzhui.top;
   
         ssl_certificate cert/lzhui.top.pem;
         ssl_certificate_key cert/lzhui.top.key;
   
         ssl_session_cache shared:SSL:1m;
         ssl_session_timeout 5m;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
         ssl_prefer_server_ciphers on;
   
         location / {
            root html/blog/;
            index index.html index.htm;
         }
      }
   
      include conf.d/*.conf;
   }
   ```

9. 创建 nginx 服务

   ```bash
   sudo vim /etc/systemd/system/nginx.service
   ```

   ```bash
   [Unit]
   Description=Nginx
   After=network.target
   
   [Service]
   Type=forking
   ExecStartPre=/usr/local/nginx/sbin/nginx -t
   ExecStart=/usr/local/nginx/sbin/nginx
   ExecReload=/usr/local/nginx/sbin/nginx -s reload
   ExecStop=/usr/local/nginx/sbin/nginx -s stop
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   ```

10. 启动 Nginx

    ```bash
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

    