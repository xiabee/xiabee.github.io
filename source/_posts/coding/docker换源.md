---
title: Docker换源
date: 2020-3-17 20:10:23
tags:
  - docker
categories:
  - coding
abbrlink: docker-source
---

## Docker相关

* [Docker入门](https://blog.xiabee.cn/posts/docker-pupy/)

* [Docker常用命令](https://blog.xiabee.cn/posts/docker-commands/)

* [Docker解决sudo](https://blog.xiabee.cn/posts/docker-sudo/)

  

## Docker换源

由于众所周知的原因，国内的访问`docker`源时会比较慢，所以我们换源到国内镜像站上。

这里依然以`Ubuntu`为例。



### 腾讯源

* [官方文档](https://cloud.tencent.com/document/product/1207/45596)

- 具体操作：

  创建加速器文件

  ```bash
  /etc/docker/daemon.json
  vim /etc/docker/daemon.json
  ```

  

  按`i`切换至编辑模式，在`daemon.json`中添加以下内容，并保存（`ESC`；输入`:`进入命令模式；`q`保存并退出）

  ```json
  {
  "registry-mirrors": [
   "https://mirror.ccs.tencentyun.com"
  ]
  }
  ```

  

  重启` Docker` 

  ```bash
  sudo service restart docker
  ```



### 阿里源

阿里源需要注册账号才能加速。

* [官网教程](https://help.aliyun.com/document_detail/60750.html)

* [注册/登录](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)后可以得到一个加速器地址
* 将加速器地址填入下面的`registry-mirrors`中：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxx.mirror.aliyuncs.com"]
}
EOF
# 创建加速器文件
sudo systemctl daemon-reload
sudo systemctl restart docker
```

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gw7yzp6zkwj30ep0a50ua.jpg)





### 清华源

* [官方文档](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gw7zfr5zwmj315q0ch44i.jpg)



### 其他源

按前面腾讯源/阿里源的换源方法，把下面地址添加到`/etc/docker/daemon.json`中：

```bash
## Docker中国区官方镜像
https://registry.docker-cn.com

## 网易
http://hub-mirror.c.163.com

## ustc 
https://docker.mirrors.ustc.edu.cn
```



```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

