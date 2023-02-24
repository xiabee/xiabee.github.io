---
title: AWS 入门
date: 2023-2-18 23:00:23
tags:
  - serverless
  - aws
categories:
  - aws
abbrlink: aws-intro
---



## 简介

* 官网：https://aws.amazon.com/cn/

> AWS 是亚马逊旗下的云计算平台，它提供了一系列云计算服务，包括计算、存储、数据库、网络、安全等等。AWS服务可以帮助个人、企业和政府机构构建和扩展他们的IT基础设施。
>
> AWS的计算服务包括 EC2，ECS，Lambda 等等，它们可以帮助用户在云上运行应用程序和处理数据。存储服务包括S3，EBS等等，用户可以在云中存储和管理数据。AWS还提供了各种数据库服务，包括RDS，DynamoDB，Redshift等等，可以帮助用户管理数据。AWS还提供了各种网络和安全服务，包括VPC，CloudFront，IAM等等，可以保护用户数据和应用程序的安全。
>
> AWS 的优势包括高可靠性、高可扩展性、灵活性、安全性等等，用户可以根据自己的需求选择使用不同的服务，灵活地扩展他们的应用程序和数据。AWS还提供了强大的监控、管理和分析工具，可以帮助用户更好地管理他们的应用程序和数据。
>
> AWS 的用户包括许多知名的公司和组织，例如Netflix、Airbnb、NASA、Unilever、[~~PingCAP~~](https://pingcap.com) 等等。



## 注册

* 打开 AWS 官网（https://aws.amazon.com/）并单击 "创建免费账户" 按钮；

* 在注册页面中，填写电子邮件地址、姓名等信息，并创建一个密码，AWS 还需要您提供信用卡信息，以便验证身份和开启账单；

![](https://s3.xiabee.cn/pic/2023/02/f04806d848e98acb56c02c9617cf21656816dad41c2a1a2b47af81096ed0f07e.png)

* 在填写完所有信息后，单击 "继续" 按钮。AWS 会要求输入电话号码以进行验证，我们可以选择接收短信或电话来进行验证

* 完成验证后，AWS 会询问是否希望使用免费套餐。注意免费套餐的使用时间，部分套餐是免费一年，部分是永久免费，超出额度按使用量收费；

* AWS 还会要求选择支持计划，个人用户建议选择最基础的支持，富哥/富婆当我没说；

* 最后，AWS 会发送一封确认邮件，按照邮件中的指示进行操作，以确认您的电子邮件地址。确认邮件中包含了一个链接，单击链接即可确认电子邮件地址；

* 完成以上步骤后，AWS 账号就创建成功了。现在我们可以使用 [AWS 控制台登录](https://console.aws.amazon.com/console/home)
* 首次登录需要使用根用户，后续再来介绍如何通过根用户设置 IAM 等，通过更安全的方式操作 AWS 账户：

![](https://s3.xiabee.cn/pic/2023/02/b9121be884fb7cc31f738b5b8bd8bc3df945a19d3024f892e0584ec615151bbe.png)

* 登录成功后将看到如下界面：

![](https://s3.xiabee.cn/pic/2023/02/f6b188ad7c289c79afea8efbe70bd3c8e4405d92a2a9d9cce2cde5597447d587.png)



## 使用与计费

AWS 是一个云平台，旨在建设一个“搭积木”服务，需要什么拿什么，一块一块的操作。例如，我当前最常用的几个云服务：

* 云存储 s3（Amazon Simple Storage Service）：用于存储备份重要数据、做图床等
* 弹性计算云 EC2（Elastic Compute Cloud）：用于部署一些主机项目，如自动代肝程序
* 专有网络 VPC（Virtual Private Cloud）：为 EC2 提供专有网络环境
* 无主机函数服务 Lambda：部署一些无主机项目，如自动签到程序
* 内容分发网络 Cloudfront：CDN，访问流量加速

AWS 大部分服务都是按使用量收费，没有基础费用。比如 s3 按照写入存储桶的内容大小收费、Cloudfront 按分发流量大小收费、EC2 按使用时长收费、Lambda 按调用次数和 CPU 时间收费等，大部分服务每月有免费额度，超出免费额度后再按量计费。



目前我使用了一个 Lambda 进行每日签到、S3 作为博客图床、Cloudfront 作为博客 CDN，调试过程中可能会调度到其他组件。当前全部组件总费用大概在两美分每月，对于我这种穷游党来说也是可以接受的水平

![](https://s3.xiabee.cn/pic/2023/02/357c5b1b7d0848745c4b6b84be986defc2e44b5e45be95395cd048d810328443.png)

（网页右上角账号信息里面可以点击 "Billing Dashboard" 查看账单）



## 安全使用

我们现在还没有创建 IAM（Identity andAccess Management）或者 SSO（Single SignOn） 用户，当前依然为 root 账户登录。

这里推荐 root 账户绑定 MFA（Multi-Factor Authentication），即开启多因子认证。开启 MFA 后，每次登录不仅需要输入密码，还需要输入 MFA  代码，防止密钥泄漏后直接被攻击者利用。开启位置在这里：

![](https://s3.xiabee.cn/pic/2023/02/e3d0b64c08edd04d7ab20fc3bd22e3d57250badbd20ee6531bb420999f1a7e15.png)



MFA 的详细信息可以参考 AWS 官方文档：https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_credentials_mfa.html

开启后，登录时会提示输入 MFA 代码：

![](https://s3.xiabee.cn/pic/2023/02/151caac3f2f05e6527df80f24babfb0a30c68c55bab9ee0e5072d52a3f249b75.png)



## 其他

关于 Lambda 签到的内容可以先看看这篇：https://blog.xiabee.cn/posts/aws-lambda/

后续我们再来介绍如何更安全的使用的 AWS。
