---
title: Git Page + Coding Page 实现负载均衡
date: 2021-11-2 22:46:18
tags:  
  - hexo
  - git
categories:
  - coding
abbrlink: git-coding-hexo
---



## 问题描述

### Github Page不能被搜索引擎收录

`Github Page`禁止了百度爬虫，导致百度无法收录我的博客；然后其他搜索引擎可能也是类似原因，均无法收录。而且曾经可以通过手动上传和`sitemap`的方式提交链接，现在连`sitemap`都读不到了......被迫转战其他平台：使用自定义域名



### Github Page在国内访问速度较慢

`github.io`经常抽风，在国内访问不稳定，考虑增加一个国内的备份。



### 初步解决方案

* 利用`coding`创建国内镜像
* 搜索引擎收入`coding`内容
* 利用可控域名，通过`CNAME`解析到不同域名中，通过`DNS`实现负载均衡、

> * 为什么是coding
> * 因为gitee实在太慢了，性能堪忧；加上[coding](https://help.coding.net/docs/pages/price.html)能白嫖六个月......



所以这期操作的前提是：**你有一个自己可控的域名**



## Github Page

* 直接利用原来的`Github Page`即可
* 还没有`Github Page`的可以看[这里](https://blog.xiabee.cn/posts/hexo-git-setup/)



## Coding

* 创建一个项目
* `Hexo`同步部署即可



### 注册账号

* [官网](https://coding.net/)



### 创建项目

* 和`Github`操作几乎一毛一样，创建一个仓库就行

  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw18swxy1kj30qf0fftc8.jpg)



### 添加SSH公钥

* [添加个人SSH公钥](https://help.coding.net/docs/project-settings/features/ssh.html)
* 添加项目SSH公钥：代码仓库->仓库设置->部署公钥（同时给该公钥写入权限）

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw18vu6zkpj31g80l248m.jpg)

注意：一定要添加`项目SSH公钥`，即使这俩一毛一样......否则会无法写入仓库。这个设计很奇怪，~被迫设置两遍~



* 添加之后`ssh -T git@e.coding.net`试一下有没有读写权限：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1901txagj30py06nn2l.jpg)



### 设置托管

* 持续部署- > 网站托管
* 这里需要实名认证，不想实名认证的同学可以直接腾讯云登录（x）

* 网站部署

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1xk5mfv7j31hc0n41kx.jpg)



稍微设置一下，之后部署成功的截图大概长这样：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw18y6sgrrj30ru0mkjwd.jpg)

此时点击访问按钮应该能访问网站，但是是`404`，因为你啥也没写。



### 设置自定义域名

我们的设计是让同一个域名指向两个不同的网址，让`DNS`服务器来判断走哪条路线最合适......所以需要一个自定义的域名——而且它原本的域名太长了，根部记不住（x）。



![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1xleyvztj31hc0n4k4w.jpg)

这里需要在域名供应商那里添加一条解析，以验证你的域名；同时需要为你的域名绑定证书。

我之前购买`xiabee.cn`的证书的时候，图便宜搞了个单域名的，现在不支持子域名证书，所以就必须重新配置证书。



本来是准备用[OHTTPS](https://ohttps.com/guide/createcertificate)搞个免费泛域名的，但是很多浏览器不认他的证书......导致我去腾讯云搞了一个新的证书（划掉）。

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1xwuzu72j311e07ltck.jpg)

想白嫖的同学可以去[OHTTPS](https://ohttps.com/guide/createcertificate)或者[letsencrypt](https://letsencrypt.org/)上面看看。



## 设置Hexo

修改根目录下的`_config.yml`最后的`deploy`：

```yml
deploy:
  - type: git
    repo: 
      github: git@github.com:xiabee/xiabee.github.io.git,main
      coding: git@e.coding.net:xiabee/xiabee-blog/xiabee.coding.me.git
      # 这里改成自己的项目地址即可
```



然后`hexo g -d`，渲染，提交。如果你的`SSH`公钥设置正确的话，应该是没有很大问题的。

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1xxmmpa0j30yf0m5dx3.jpg)



## 设置域名解析

最后把自定义域名同时解析到两个记录值即可：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1xz4983aj30kg031jry.jpg)



## 国内外访问测试

国内访问指向`coding page`：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1y9u3pagj30js05i77q.jpg)



国外访问指向`Github Page`：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw1y8qwz28j30ib05bn0b.jpg)



此时我们的分布式站点已经配置好了（✌）
