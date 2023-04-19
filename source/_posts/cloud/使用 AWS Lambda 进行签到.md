---
title: 使用 AWS Lambda 进行签到
date: 2022-12-11 12:00:23
tags:
  - cloud
  - aws
categories:
  - aws
abbrlink: aws-lambda
---

<meta name="referrer" content="no-referrer">

# 背景

* 很久之前做了个[基于 AWS Lambda 的 Gladows 的自动签到脚本](https://github.com/xiabee/aws-glados-checkin)，一直没有写使用教程，今天来补充一下（不是我鸽，是真的太忙了.jpg）
* 账号创建啥的太久远了已经忘记怎么操作的了，记忆中不是很难，按照 AWS 官方提示一步一步走就好了

* 签到架构图：

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h7hz51nsbbj31pq0rmty1.jpg)

* Glados 签到过程分析如下：https://blog.xiabee.cn/posts/glados-intro/ ，想二创的朋友可以看看这篇

## 什么是 AWS Lambda

* AWS：Amazon Web Service，中文名叫亚马逊云服务平台，一个做云服务的厂商提供的云服务，类似于国内的阿里云、腾讯云等
* Lambda：AWS 提供的一项云服务，是一种 serverless 服务，在不提供完整服务器的前提下让客户实现云计算功能

## 为什么是 AWS & Serverless

* Serverless：便宜。选择 serverless 是因为我们没有确切的存储需求，只做签到的话，完全不需要硬盘等设备，只做计算就可以了，这种情况下 serverless 会比 EC2 / VPS 便宜很多；
* AWS：云服务商很多，但是钱多到可以烧的服务商有限......AWS 是一个对小客户比较友好的一家，完全实现按量收费，绝大多数服务没有基础费用。而且 AWS Lambda 每个月都有很大一定量的免费额度，至少我们这个签到脚本几乎可以一直免费使用，不像某讯云/某里云免费六个月就要开始收基础费用了......



如图所示：仅使用 Lambda 进行签到，一个月产生的费用

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h9281y94e0j32780so47z.jpg)



# 准备工作

* 最好有一张 Visa 卡，因为 AWS 账号的时候需要绑定银行卡，但是银联应该不行（不记得 Paypal 行不行了，我只有 Visa 所以绑定的是 Visa）
* 创建一个 AWS 账号，直接创建 root 账号就行，根据官方提示一步一步走，理论上不难（如果你能顺利通过人机验证的话......当时我的验证码一直猜不对）

* 从安全合规的角度说，使用 root 账号进行配置有安全风险，但是个人用户嘛，暂时先不管那么多，能保证自己的密码 / MFA 不泄漏就已经很不错了......等熟练使用 AWS 之后再来提升安全配置。



# 创建 lambda 函数

## 进入模块

* 在控制台左上角搜索 `Lambda`，找到该功能并进入：

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h928co6a3mj31o00s4an7.jpg" alt="image.png" style="zoom:100%;" />



* 找到右上角 创建函数，如图

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h928e6c6bvj30ti080gms.jpg" alt="image.png" style="zoom:100%;" />



## 运行时设置

* 注意，因为我们[代码](https://github.com/xiabee/aws-glados-checkin)用的是 `golang`，所以运行时选择 `Go 1.x`
* 由于 `Lambda` 不支持 `Arm` 的 `go` 运行时，所以我们选择 `x86` 架构，在最后编译过程中用 `x86` 进行编译即可
* 我们代码编译生成的程序是 `main`，所以这里的处理程序也设置为 `main`

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h928sowu2aj31bq0uc7an.jpg" alt="image.png" style="zoom:100%;" />



* 选择 arm 的报错提示：

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h928l2lexij31lq060tbk.jpg)



## 编译与上传

* 这里我们必须使用 x84 架构进行编译：

```bash
# Remember to build your handler executable for Linux!
GOOS=linux GOARCH=amd64 go build -o main main.go
zip main.zip main
```



* 然后把 `main.zip` 上传至 AWS

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h928mnajx0j32qo16ywvi.jpg)



## 代码测试

发送空数据包，直接测试代码是否能够运行：正常情况下这里是能执行成功的，但是返回应该是一个“未登录”之类的信息，因为确实没有输入登录信息

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h928w44oe3j32jo110k3v.jpg)



## 设置登录信息

* 这个签到脚本是利用 `cookie` 进行登录的，然后通过 `Wechatbot` 的 `webhook` 来发送消息，这里不详细解释二者原理了，只介绍一下怎么设置。
* 代码中通过环境变量来读取上述两个值

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h9294fbeuuj30jq0460ts.jpg)

* 这里我们把 `COOKIE` 和 `WECHAT_KEY` 的键值对写进 AWS Lambda 的环境变量中，注意大小写
  * 注意，把敏感信息写入环境变量不是 AWS 最佳安全实践的做法，如果有条件的话，最好使用 KMS 进行操作，防止 Lambda 角色账号被拿下后，攻击者能够直接看到更高级别的敏感信息；
  * 但是我们已经是使用 root 账号登录的了......如果这个账号被拿下了基本上没救了（划掉）

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h92921an2yj31aa0l2wkb.jpg" alt="image.png" style="zoom:100%;" />



* 不知道如何获取 cookie 的朋友可以看看 [GitHub 的另一个仓库](https://github.com/xiabee/glados-checkin#cookie-%E8%8E%B7%E5%8F%96%E6%96%B9%E5%BC%8F)
* wechatbot 非必须，想加的话可以参考[这篇博客](https://blog.xiabee.cn/posts/wechat-bot/)



设置完成后再次进行代码测试，应该会在日志中看到签到信息。



# 设置触发器

签到功能，那当然得是每天自动执行啊......AWS 提供一个名为 `EventBridge` 的触发器，可以设置定时执行

## 创建触发器

* 直接 添加触发器（这里我已经添加好了，就不重复添加了）

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h929bjj5f5j317y0ocq6v.jpg" alt="image.png" style="zoom:100%;" />



* 根据提示完成基础设置，最后设置 “规则”，“创建规则”

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h929dcp93lj32vc0zmaps.jpg)



## 设置触发规则

* 计划模式，在特定时间触发。填写完成后会有详细的提示，告诉你 Cron 表达式所表示的具体日期，对 Cron 小白也很友好（划掉）

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h929ewv9iaj31r2170tng.jpg" alt="image.png" style="zoom:100%;" />



## 选择目标

* 将目标设置为刚刚创建的 Lambda 函数

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h929hnoc0nj31pi16yncf.jpg" alt="image.png" style="zoom:100%;" />



此时全部设置已经完成，最终架构如下：

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xgy1h929j6v3f2j31o40rkdk9.jpg)



# 小结

* Serverless 可以做的事情很多，这里只是简单举了个签到的例子
* AWS 按量付费，免费额度比较多，适合个体发烧友
* 本文使用 root 账号登录、环境变量中存储敏感信息不符合 AWS 最佳安全实践......但是就这样吧，以后再来写如何安全且规范的使用 AWS
