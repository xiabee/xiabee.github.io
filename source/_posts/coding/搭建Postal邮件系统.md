---
title: Docker 搭建 Postal 邮件系统
date: 2022-5-8 10:00:23
tags:
  - postal
  - docker
categories:
  - coding
abbrlink: postal-mailserver
---

## 前情提要

[上次](https://blog.xiabee.cn/posts/hydro-docker/)搭建了一个`Hydro-OJ`，注册需要邮箱验证，想着要不一步到位顺带把**邮件系统**也处理了（x）

## 系统选择

有很多酷炫且开源的邮件系统，`github`上能搜到很多，比如[mailcow](https://mailcow.email/)，[postal](https://docs.postalserver.io/)等。本片主要介绍`postal`的搭建方式。

![](https://docs.postalserver.io/screenshot.png)

## Postal 简介

* 官网：[postal](https://docs.postalserver.io/)

* 项目地址：[GitHub - postalserver/install](https://github.com/postalserver/install)

> **Postal** is a complete and fully featured mail server for use by websites & web servers. Think Sendgrid, Mailgun or Postmark but open source and ready for you to run on your own servers. Postal was developed by [Krystal](https://k.io/) to serve its own mail processing requirements and we have since decided that it should be released as an open source project for the community.

摘自官网，懒癌晚期，就不翻译了（x）

本篇主要讲解如何使用`docker`进行安装，即如何利用[官方推荐](https://docs.postalserver.io/)的安装方式安装`postal`邮件系统。

本篇均以`Ubuntu/Debian`系统为例，其他包管理器的系统可以做类似参考——因为主要工作是容器做的，所以和系统版本不是直接相关。

## 准备工作

### 服务器配置

官方强烈建议服务器至少拥有以下配置：

- 至少 `4GB` `RAM`内存
- 至少 `2 `个`CPU` 核心
- 至少 `100GB` 的硬盘空间

但是实测之后发现，如果服务体量不大的话，`2~3G`内存也是基本够用的，硬盘空间也不需要`100G`，大硬盘只是用于存储邮件和备份。

另外，邮件系统需要一个可控域名，即需要一个域名，同时需要能够设置子域名。

### 系统环境

```bash
sudo apt install git curl jq
# 安装所需软件

git clone https://postalserver.io/start/install /opt/postal/install
# 克隆安装代码

sudo ln -s /opt/postal/install/bin/postal /usr/bin/postal
# 创建软连接，用于快速启动postal
```

如果上述命令不能正常执行，可以尝试添加`sudo`，或者将代码克隆到常规目录下，再利用`sudo cp`等方式复制到对应目录下。

### Docker

服务器中已经安装了`docker`，并且能够以非`root`身份运行。

```bash
sudo apt install docker
# 安装docker

sudo gpasswd -a ${USER} docker
newgrp - docker
# 切换到docker组

sudo service docker restart
# 重启服务，确保非root用户可以运行docker
```

* 不会安装的话可以参考[官方文档](https://docs.docker.com/get-docker/)

* 如果不能以普通用户运行`docker`的话可以参考这篇博客：[不输入sudo运行Docker · xiabee-瞎哔哔](https://blog.xiabee.cn/posts/docker-sudo/)

### MariaDB

```bash
docker run -d \
   --name postal-mariadb \
   -p 127.0.0.1:3306:3306 \
   --restart always \
   -e MARIADB_DATABASE=postal \
   -e MARIADB_ROOT_PASSWORD=postal \
   mariadb
```

这里直接利用`docker`运行所需数据库，其中数据库设置为`postal`，`root`密码设置为`postal`；如需修改，直接将`MARIADB_DATABASE=xxx`和`MARIADB_ROOT_PASSWORD=xxx`修改掉就行，之后的配置中如果遇到连接服务器的服务，改为对应密码即可。

### RabbitMQ

```bash
docker run -d \
   --name postal-rabbitmq \
   -p 127.0.0.1:5672:5672 \
   --restart always \
   -e RABBITMQ_DEFAULT_USER=postal \
   -e RABBITMQ_DEFAULT_PASS=postal \
   -e RABBITMQ_DEFAULT_VHOST=postal \
   rabbitmq:3.8
```

同样，利用`docker`运行消息队列。需要改密码的话将`RABBITMQ_DEFAULT_PASS=xxx`等改掉即可。

### docker-compose 2

`postal`需要利用`2.0`以上的`docker-compose`进行安装。

查看`docker-compose`版本：

```bash
docker-compose version
# Docker Compose version v2.4.1
```

如何安装高版本的`docker-compose`可以参考这篇博客：[安装 docker-compose 2 · xiabee-瞎哔哔](https://blog.xiabee.cn/posts/docker-compose-2/)

## 安装过程

这里假设我们使用的域名为`yourdomain.com`

### 初始化 postal

```bash
postal bootstrap postal.yourdomain.com
```

在初始化之后，需要将`postal.yourdomain.com`解析到服务器的`ip`上，如果在本地搭建则解析到`localhost`或者`127.0.0.1`上。

同时，上述命令会在 `/opt/postal/config`中生成三个文件：

- `postal.yml` ：主要配置文件
- `signing.key` 各项签名的私钥
- `Caddyfile` `Caddy`服务器的配置文件（`Caddy`类似于`Nginx`，用作代理服务器）

### 初始化数据库

```bash
postal initialize
postal make-user
```

创建第一个用户，执行命令过程中会提示你设置用户名和密码。

### 运行 postal

```bash
postal start
```

此时会开始运行容器，如果没有相关镜像会直接拉取，耐心等待即可。

注意：需要利用`2.0`以上的`docker-compose`进行安装，确保宿主机中的`docker-compose`版本高于`2.0`

### Caddy (可选)

```bash
docker run -d \
   --name postal-caddy \
   --restart always \
   --network host \
   -v /opt/postal/config/Caddyfile:/etc/caddy/Caddyfile \
   -v /opt/postal/caddy-data:/data \
   caddy
```

`Caddy`为代理服务器，可选项很多，比如`Nginx`，`Appache`等，甚至可以不装（如果不需要`https`的话）

最终应该看到这个样子：

```bash
postal status
```

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1h218be8ztcj31jk0ss4qp.jpg)

## 注意事项

上述方法安装的`postal`服务端运行在`5000`端口，如果浏览器访问不了的话检查防火墙是否放行该端口的流量。

如果还是不能访问，检查`/opt/postal/config`目录下的`postal.yml`，找到`web_server`模块：

```yml
web_server:
  # Specify configuration for the Postal web server
  bind_address: 127.0.0.1
  port: 5000
```

原文件默认将地址绑定到`127.0.0.1`，只能在域内访问，从域外访问就会被拦截。如果`VPS`没有特定设置的话，直接把绑定地址设置为`0.0.0.0`，允许任何流量即可：

```yml
web_server:
  # Specify configuration for the Postal web server
  bind_address: 0.0.0.0
  port: 5000
```

同时，如果没有设置`SSL`证书的话，可以把服务协议改为`http`

```yml
web:
  # The host that the management interface will be available on
  host: postal.yourdomain.com
  # The protocol that requests to the management interface should happen on
  protocol: http
```

## 最终效果

浏览器访问`http://postal.yourdomain.com:5000`，即可看到以下界面：

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1h218lwhhhqj30e10aht9u.jpg" title="" alt="image.png" data-align="center">

到目前为止，`postal`已经搭建完毕，具体的设置我们下期再写。

## Refference

> https://docs.postalserver.io/
