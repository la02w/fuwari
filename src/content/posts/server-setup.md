---
title: 服务器初始化
published: 2024-09-06
description: ""
image: ""
tags: []
category: ""
draft: false
---

# 云服务器服务器初始化

## 创建普通 bash 用户

```bash
useradd -m -s /bin/bash 新用户名
passwd 新用户名
```

- -m：创建用户的同时创建用户的家目录。
- -s /bin/bash：指定用户登录时使用的 shell，这里指定为 bash。

### 设置 sudo 权限

```bash
visudo

# 在 sudoers 文件中找到如下类似的一行：
# root    ALL=(ALL:ALL) ALL
# 在这一行下面，添加以下内容来授予用户 sudo 权限：
user    ALL=(ALL:ALL) ALL
```

## 设置主机名

```bash
sudo hostnamectl set-hostname 主机名
# 刷新终端
bash
```

## 更新软件包

```bash
sudo apt update
sudo apt upgrade
```

### 切换软件镜像源

> 快速配置

```bash
# 使用sudo用户或者手动复制
cat <<'EOF' > /etc/apt/sources.list
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirror.nju.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirror.nju.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirror.nju.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirror.nju.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirror.nju.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirror.nju.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
# deb https://mirror.nju.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# # deb-src https://mirror.nju.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirror.nju.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirror.nju.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
EOF
```

## 安装 docker

### 添加 docker 密钥

```bash
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 添加阿里云 docker 软件源

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] http://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 更新软件源

```bash
sudo apt update
```

### 安装 docker

```bash
sudo apt install docker-ce
```

### 配置 docker 用户

```bash
sudo usermod -aG docker $USER
# 更新用户组
newgrp docker
```

## 配置 docker

### 设置镜像加速

```bash
sudo vim /etc/docker/daemon.json
```

```json
{
  "registry-mirrors": [
    "https://hub.uuuadc.top",
    "https://docker.anyhub.us.kg",
    "https://dockerhub.jobcher.com",
    "https://dockerhub.icu",
    "https://docker.ckyl.me",
    "https://docker.awsl9527.cn"
  ]
}
```
