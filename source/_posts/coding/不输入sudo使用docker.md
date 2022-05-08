---
title: 不输入sudo运行Docker
date: 2020-3-19 20:00:23
tags:
  - docker
categories:
  - coding
abbrlink: docker-sudo
---

`docker`对权限要求较高，需要`sudo`权限才能运行，但是每次敲命令都加`sudo`就显得很累赘，这里有个化简办法：**将用户加入docker组，实现不加sudo执行docker命令**

## 查看docker组

查看/`etc/group`,确定是否存在docker组

```bash
 cat /etc/group | grep docker
```

安装Docker后，docker组已经创建好了，所以上面命令的输出为： `docker:x:120:ubuntu `

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw7zwhd5yxj30cb031q3t.jpg)

## 将当前用户添加到 docker 组

```bash
 sudo gpasswd -a ${USER} docker
```

## 重新登录或切换到docker组

```bash
newgrp - docker
# 切换到docker组

sudo service docker restart
# 重启服务
```

## 检查效果

 不加sudo直接执行docker命令检查效果：`docker images`

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h2178op16jj30lp09faha.jpg)

~~执行成功，妈妈再也不用担心我敲 sudo 啦~~
