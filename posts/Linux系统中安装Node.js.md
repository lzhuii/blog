---
title: 'Linux 系统中安装 Node.js'
date: '2023-11-30 08:56:00'
---

1. 下载 Node.js

   ```bash
   wget https://nodejs.org/dist/v20.10.0/node-v20.10.0-linux-x64.tar.xz
   ```

2. 解压 Node.js

   ```bash
   tar xvf node-v20.10.0-linux-x64.tar.xz -C /usr/local
   ```

3. 配置环境变量 `/etc/profile`

   ```bash
   # NODE
   export NODE_HOME=/usr/local/node-v20.10.0-linux-x64
   export PATH=$PATH:$NODE_HOME/bin
   ```

4. 刷新环境变量

   ```bash
   source /etc/profile
   ```

5. 验证

   ```bash
   node -v
   npm -v
   ```

