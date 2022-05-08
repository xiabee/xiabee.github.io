---
title: 安装 docker-compose 2
date: 2022-5-8 12:00:23
tags:
  - postal
  - docker
  - docker-compose
categories:
  - coding
abbrlink: docker-compose-2
---



## 前期提要

[上期](https://blog.xiabee.cn/posts/postal-mailserver/#/docker-compose-2)我们讲到，`postal`的安装需要高版本的`docker-compose`。



一般情况下，我们直接使用包管理安装即可，但是我在安装过程中遇到了包管理器中没有高版本的情况......所以写下本篇记录一下。



本篇依然以`Ubuntu/Debian`为例。



## 查看版本

```bash
docker-compose -v
```

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h219bl0sbaj30d202uq3y.jpg)

此时看到我们的版本是`1.25.0`，是低于`2.0`的，无法运行`3.9`及以上的`docker-compose.yml`



## 安装方式（一）

* 可以参考[官网](https://docs.docker.com/compose/install/)

* 这里使用一种比较暴力的安装方式：直接手动下载可执行文件



### 查看最新版本

浏览器访问`docker-compose`的[代码仓库](https://github.com/docker/compose/releases)（可能需要翻墙）

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h219l8awe5j30vs0gv7c4.jpg)

截至目前，最高版本是`2.5.0`。



![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h219mrogiqj30fc097gp4.jpg)



### 下载文件

然后从发布的二进制文件中，找到对应系统的可执行文件，下载下来：

```bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```

上述命令为`$HOME`目录下的活动用户安装`Compose V2`。



如果需要为系统中的所有用户安装`Docker Compose V2`的话则执行：

```bash
mkdir -p /usr/local/lib/docker/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
```



### 添加权限

```bash
chmod +x ~/.docker/cli-plugins/docker-compose
```



### 测试安装

```bash
docker compose version
docker-compose version
# 注意此时的两个结果
```

如果上述两个命令执行结果不同，说明宿主机中已经存在了低版本的`docker-compose`，而刚刚安装的高版本`compose`仅作为`docker`的插件在使用，没有改变`/bin`中的可执行文件。此时直接安装`postal`依然会失败的......



那么就需要更暴力的安装方式：直接替换低版本



## 安装方式（二）

直接替换原来的`docker-compose`二进制文件。

### 

### 查看最新版本

同上，浏览器访问官网查看即可。



### 下载文件

同上，可以直接手动下载上传给服务器，也可以利用`curl`下载：

```bash
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o docker-compose
```



### 添加权限

```bash
chmod +x docker-compose
```



### 查找文件位置

```bash
find / -name docker-compose
```

一般来说在`/usr/bin/docker-compose`中：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h21a2ndezyj30g9041tbs.jpg)



### 替换文件

```bash
sudo cp docker-compose /usr/bin/docker-compose
```



### 测试安装

```bash
docker-compose -v
```

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h21a4o9fazj30ef03fq3x.jpg)

此时已经升级到高版本了。


