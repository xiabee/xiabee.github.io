---
title: Mac 环境配置(4)——BurpSuite 破解
date: 2022-9-30 12:00:23
tags:
  - mac
  - ctf
categories:
  - coding
abbrlink: mac-init
---



## 简介

* [Burp Suite](https://portswigger.net/burp)：~~懂得都懂，不解释了~~
* 一款非常流行的应用层抓包工具，做 `WEB` 安全的人应该都见过，至少都听说过



## 下载与安装

`Burp Suite` 下载地址：https://portswigger.net/burp/releases

* `Windows` : 需要下载`jar` 文件，通过启动器同时启动注册机和主体文件

* `Mac` : 直接官网下载官方安装包进行安装即可，后文介绍如何注入注册机

注册机下载地址：https://github.com/h3110w0r1d-y/BurpLoaderKeygen



## 注入注册机

### 移动注册机

* 将下载的注册机 `BurpLoaderKeygen.jar  `放到 `Burp Suite` 的`app` 目录内

* 具体位置：`/Applications/Burp Suite Professional.app/Contents/Resources/app`

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7huy5a9qkj31s60um7lh.jpg" alt="image.png" style="zoom:67%;" />



### 修改 vmoptions

添加以下内容：

```bash
-Dfile.encoding=utf-8
-noverify
-javaagent:BurpLoaderKeygen.jar
 -Xmx2048m
```

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7hv0cy0spj312c0o0tye.jpg" alt="image.png" style="zoom:67%;" />



### 运行注册机

添加完成后，打开终端，先使用注册机加载 BP 运行注册一下：

```bash
cd "/Applications/Burp Suite Professional.app/Contents/Resources/app"
"/Applications/Burp Suite Professional.app/Contents/Resources/jre.bundle/Contents/Home/bin/java" -jar BurpLoaderKeygen.jar
```



注册完成后就可以应该直接正常启动了（如果遇到系统信任的问题，进系统设置里面信任即可）

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h7hv6v7wdkj312y0ac439.jpg)



## 其他配置

`SSL` 证书等问题暂时先不写了，有空再来更新（划掉）

也可以直接参考[国光大佬](https://www.sqlsec.com/2019/11/macbp.html#Firefox-Dev)的博客（膜）



## Reference

> https://www.sqlsec.com/2019/11/macbp.html
>
> https://github.com/h3110w0r1d-y/BurpLoaderKeygen
