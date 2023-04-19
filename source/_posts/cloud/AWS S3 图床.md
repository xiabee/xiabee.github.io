---
title: AWS S3+CloudFront 打造自己的图床
date: 2023-4-9 10:00:23
tags:
  - cloud
  - aws
  - s3
categories:
  - aws
abbrlink: aws-pichost
---



## 背景

图床是一个托管博客图片的地方，常见图床有新浪微博、七牛云、SM.MS 等。

但是天下没有免费的午餐，免费图床总有各种问题（比如我的[微博图床抢救](https://blog.xiabee.cn/posts/sina-pichost/)），于是决定自己建一个图床。这里使用的是 AWS 的 S3 和 CloudFront。



### 为什么选择 AWS S3

* 国内云的带宽费用普遍高于海外云
* 相比于国内云，AWS 的合规性更好

### 为什么不使用 CloudFlare

* 目前体量较小，S3 更实惠

![image-20230409141703032](https://s3.xiabee.cn/pic/2023/04/9b5e724e3ef4779181b00c79aa823795605baadd5eb388f10ed176c008c1add9.png)



## 简介

### AWS S3

>  AWS S3 是一个弹性、高扩展的对象存储服务。它可以存储任何类型的数据，包括图片、音频、视频、文档等等。用户可以在 AWS 控制台中轻松地创建和管理存储桶，上传和下载数据，进行权限管理和数据备份等操作。S3 的可用性和耐久性非常高，可以保证数据的安全和可靠性。



### AWS CloudFront

> AWS CloudFront 是一个全球内容分发网络，可以帮助用户加速静态和动态内容的分发。用户可以将静态和动态内容存储在 S3 中，并通过 CloudFront 分发到全球各地的用户。CloudFront 可以自动缓存内容，提高访问速度，同时保证内容的安全性和可靠性。用户可以在AWS控制台中轻松创建和管理分发，设置缓存策略、安全性控制和报告等。



综上，我们利用 S3 作云存储，利用 Cloudfront 做 CDN（内容分发网络），实现一个安全、可靠、高速的图床。



## 步骤概述

1. 创建一个 AWS 账号
2. 在 AWS 中创建一个 S3 bucket
3. 创建 cloudfront distribution 并绑定 S3 bucket
4. 创建 DNS 记录，绑定 cloudfront 自定义域名



## 详细步骤

接下来详细介绍一下

### 注册账号

[之前的博客](https://blog.xiabee.cn/posts/aws-intro/)中有相关介绍，这里我们注册的是 Global 账号。

> Q：能不能注册中国区账号
>
> A：亚马逊中国审核比较严格，考虑到备案等因素，建议使用全球亚马逊



> Q：全球亚马逊是否必须使用外汇卡
>
> A：之前必须使用 Visa，现在也可以使用人民币

![image-20230409143503642](https://s3.xiabee.cn/pic/2023/04/591104855a25d1449c9caf3c79c07e8940f7a6ad6fa5d405205690972cfca254.png)



### 创建 S3 存储桶

在控制台主页搜索 “S3”，并进入 S3 控制台：

![image-20230409143627593](https://s3.xiabee.cn/pic/2023/04/084c7afa058abe5c657dc7f9a64dc58c94f504beb40865005f30e7783c91dd72.png)



在存储桶控制台选择 “创建存储桶”：

![image-20230409143806323](https://s3.xiabee.cn/pic/2023/04/4c123fdad650a1bd4be6378b731937f3046ee0456b500ee190143f4b0b87f46e.png)



在存储桶权限相关设置的位置，如果有不清楚的地方，就直接按照 AWS 给出的推荐进行设置，比如开启 “阻止所有公开访问权限”：

![image-20230409144002387](https://s3.xiabee.cn/pic/2023/04/a0bdd0e77ff583201c53b5db2225b1f912764cfdf454697affd20253caa6879f.png)



关闭存储桶公网访问，所有的公网流量全部走 Cloudfront，很大程度上保护了存储桶安全。



### 创建 Cloudfront 内容分发网络

控制台主页搜索，进入 cloudfront 控制台：

![image-20230409144239242](https://s3.xiabee.cn/pic/2023/04/88173e2b6af406026808a9b1962d348ee169e61274282acf0c9dce923feec062.png)



#### 创建分配

![image-20230409144315962](https://s3.xiabee.cn/pic/2023/04/beca56b1b3501b035654ba6cfd5add24d9fd4e3a02064239e3a3612890663280.png)





#### 分配源域

创建分配时第一步要选择 “源”，默认会给出 S3 的源域，直接选择即可：

![image-20230409144447962](https://s3.xiabee.cn/pic/2023/04/8ad2fbd398e0aacdd492c17dd21aac3efb9e7d4647caa961f889fe5386b69e85.png)



#### 设置来源身份访问

前面我们设置了禁止存储桶公网访问，这里我们就使用 “来源访问控制设置” 或者 “Legacy access identities” 进行访问控制（途中第二项或者第三项），不要直接公开存储桶。

![image-20230409144724030](https://s3.xiabee.cn/pic/2023/04/6a4ecd7b20ba62db99f830478e9624973b704797843097df1c75b3a2eb7f7c93.png)



后续设置和分发性能相关，按需设置即可



#### 自定义域名

找到相关分配 ➡️ 编辑设置 ➡️ 备用域名

注意，使用自定义域名时，需要把自定义的域名 CNAME 解析到原有的 cloudfront 域名中，否则会设置失败

![image-20230409145227527](https://s3.xiabee.cn/pic/2023/04/35e4560b97fd097301260d013fe76ea3296d6b663d328720122a3679dc41a9e8.png)



#### 自定义 SSL 证书

往下拉一点就会有证书选项，自己没有证书的话直接请求公网证书即可。注意这里也需要对域名进行 CNAME 解析验证，按照 AWS 提示进行相关设置即可。

![image-20230409145457368](https://s3.xiabee.cn/pic/2023/04/f9286393d96534649540c6e02a60ed004172acca4b8015af42760cedd71ad1d4.png)



## 验证

完成上述步骤后，S3 应该为禁止公网访问：

![image-20230409150101612](https://s3.xiabee.cn/pic/2023/04/790235a8871138930f18d809539d210220fa55ce6e975f9806c147be32ac620b.png)



Cloudfront 有已分配的域名和备用域名：

![image-20230409150027062](https://s3.xiabee.cn/pic/2023/04/6a444a164f713aae94ccd771ea21e0fc6ba6200b05007ee1506aa4949c95304e.png)



此时我们上传一张图片到 S3 中，直接访问 [S3 的 url](https://xiabee-storage-host.s3.ap-southeast-1.amazonaws.com/pic/wallpaper/3.jpg) 是不行的：

![image-20230409150225912](https://s3.xiabee.cn/pic/2023/04/77bf9f0cb50d25a1e9b75f20b81f31784a8c87763f4a6f5b8ca799b438517864.png)

换成 [cloudfront 的域名](https://s3.xiabee.cn/pic/wallpaper/3.jpg)，可以访问：

![image-20230409150401331](https://s3.xiabee.cn/pic/2023/04/175adcbf345c6e7d2d3c33ccf8112337954792cebb27d158cef8b9f9e23bdf82.png)



此时我们图床的设置已经完成辣！



## 其他设置

关于 Typora 编写 Markdown 快速插图的问题，可以使用 [PicGO](https://github.com/Molunerfinn/PicGo) 等插件。

个人感觉体验还行，支持剪贴板直接上传，具体使用看[这里](https://blog.xiabee.cn/posts/pichost-picgo/)
