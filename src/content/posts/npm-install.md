---
title: NPM反向代理安装配置教程
published: 2024-09-11
description: "反向代理安装配置教程"
image: "https://cdn.la02.cc/pichub/2024/09/11/1726017896.png"
tags: [Docker, Web项目, Linux]
category: 指南
draft: false
lang: ""
---

## 前言

:::tip
如果有哪部分不清楚，不明白，可以发送相关问题到admin@la02.club反馈，助力博主完善文章。
:::

## 安装 docker

使用阿里云[安装 docker](/posts/server-setup/#安装-docker)，并设置加速源。

## 安装 nginx-proxy-manager

创建 `docker/npm` 目录，我们选择在家目录创建 `~/docker/npm`

```bash
mkdir ~/docker/npm & cd ~/docker/npm
```

### 创建 docker-compose.yml

```bash
vim docker-compose.yml
```

编写 `docker-compose` 配置信息

```yml
services:
  app:
    image: "chishin/nginx-proxy-manager-zh:release"
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - "80:80" # Public HTTP Port
      - "443:443" # Public HTTPS Port
      - "81:81" # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP

    # Uncomment the next line if you uncomment anything in the section
    # environment:
    # Uncomment this if you want to change the location of
    # the SQLite DB file within the container
    # DB_SQLITE_FILE: "/data/database.sqlite"

    # Uncomment this if IPv6 is not enabled on your host
    # DISABLE_IPV6: 'true'

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

:::tip[提示]
开放 80 443 端口\
Web 站点在 81 端口\
web 站点的默认用户名密码：

- 用户名：admin@example.com
- 密码：changeme
  :::

### 拉取镜像并启动

```bash
docker compose up -d
```

### 登录并配置第一个反代 npm.xxx.com

访问 web 站点，ip:81

:::tip[提示]
在终端输入 `curl ip.sb` 获取公网 ip

使用默认账号密码登录之后要设置新邮箱和密码
:::

#### 配置 DNS 域名解析

添加两个域名解析 `@`和`npm`，解析到对应的公网 ip 地址

![picture 0](https://cdn.la02.cc/pichub/2024/09/11/1726016619.png)

cloudflare 配置为例，添加 DNS 解析，配置`A记录类型`，`@主机记录`，`IP地址`，关闭 CF 代理。

`npm`同理，修改`@`为对应的名称

#### 进入 NPM 创建反向代理，并获取免费证书开启 https

主页仪表盘页面，选择`代理服务` -> 右上角的`添加代理服务`

![picture 2](https://cdn.la02.cc/pichub/2024/09/11/1726017205.png)

域名为 dns 解析的二级域名，此处为`npm.xxx.com`，协议为默认 `http` 即可，设置对应的 `ip` 地址和`端口`

![picture 8](https://cdn.la02.cc/pichub/2024/09/11/1726019856.png)

配置 SSL ，`申请新的SSL证书`，`开启SSL` `同意条款即可`，之后保存即可。会等待一会儿获取 SSL 证书。

![picture 3](https://cdn.la02.cc/pichub/2024/09/11/1726017487.png)

`状态`，`SSL`正常显示即可点击左侧地址访问，国内 IP 需要 ICP 备案信息

![picture 4](https://cdn.la02.cc/pichub/2024/09/11/1726017640.png)

### 泛域名证书

#### 生成泛域名证书

[查看教程](/posts/acme/)

#### 上传到反向代理服务器

点击`SSL证书` --> `添加SSL证书` --> `上传证书`

![picture 6](https://cdn.la02.cc/pichub/2024/09/11/1726018436.png)

有三个必填项

![picture 7](https://cdn.la02.cc/pichub/2024/09/11/1726018807.png)

|  名字\*  |   密钥\*   |   证书\*   |   中间证书    |
| :------: | :--------: | :--------: | :-----------: |
| \*.xx.xx | domain.key | domain.cer | fullchain.cer |

最后编辑代理服务的 SSL 证书来源，配置上传的证书文件即可。
