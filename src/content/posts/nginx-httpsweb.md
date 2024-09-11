---
title: 使用nginx配置一个HTTPSWeb站点
published: 2024-09-12
description: "使用nginx配置一个HTTPSWeb站点"
image: ""
tags: [Linux, Docker, Web项目]
category: "指南"
draft: false
lang: ""
---

:::tip[注意]
此文章代码顺序与视频不一致，请仔细查看视频中使用的代码作用
:::

创建目录并进入

`mkdir -p docker/nginx/{ssl,html,conf.d} & cd docker/nginx/`

创建 docker-compose.yml 文件

`vim docker-compose.yml`

写入配置

```yml
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./ssl:/etc/nginx/ssl
      - ./html:/usr/share/nginx/html
      - ./conf.d/:/etc/nginx/conf.d/
    restart: always
```

创建 `nginx` 配置文件和 `html` 首页文件

```bash
touch conf.d/1.conf
echo "123123" > html/index.html
```

为 nginx 安装证书

```bash
acme.sh --install-cert -d example.com \
--key-file       /home/ubuntu/docker/nginx/ssl/server.key  \
--fullchain-file /home/ubuntu/docker/nginx/ssl/server.cer
```

编写 nginx 配置文件

```conf
server {
    listen       80;
    listen       443 ssl;
    listen  [::]:80;
    listen  [::]:443 ssl;
    server_name  example.com;

    ssl_certificate /etc/nginx/ssl/server.cer;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;

    add_header Strict-Transport-Security "max-age=31536000" always;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

`docker compose up` 启动测试，需要后台运行可以使用`docker compose up -d`
