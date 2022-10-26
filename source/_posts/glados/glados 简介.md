---
title: Glados 签到简介
date: 2022-10-20 12:00:23
tags:
  - glados
categories:
  - coding
abbrlink: glados-intro
---

## GLADOS 是什么

简单来说是一个代理池供应商，具体作用为隐藏真实IP、防溯源等，是安全从业者必不可少的工具之一。

* 项目地址：[Github](https://github.com/glados-network/GLaDOS)
* 注册地址：在项目地址中时时更新
* 注册邀请码：`EEXLM-SZMDH-NJT3P-9NA4V` （你用我的邀请码咱俩都可以多白嫖一天）



## 为什么是 GLADOS

因为白嫖很快乐。

* `GLADOS` 有[教育计划](https://glados.rocks/console/education)，如果你还没有毕业，可以用学生邮箱免费试用一年
* 如果是付费用户，每日签到可以延长一天会员时间，理论上说坚持签到可以一直免费续杯（划掉）

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j2sv71fzj31sw0osdmk.jpg" alt="image.png" style="zoom:67%;" />



## 如何进行签到

其实本文只是想介绍一下常见 `API` 的分析方法，并没有摁推的意思。这里我们主要通过 [BurpSuite](https://blog.xiabee.cn/posts/mac-bp/) 进行签到抓包，分析具体的请求过程，为后续编写自动化签到程序做铺垫。



### 手动签到

进入[主页](https://blog.xiabee.cn/posts/mac-bp/) ➡️ （登录） ➡️ 首页（往下翻） ➡️ 我的会员 ➡️ 会员签到，然后就发现签到成功啦

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j30b0kfcj30z40lqjvr.jpg" alt="image.png" style="zoom:67%;" />



### 签到分析

#### 请求流程

打开 `BurpSuite`，拦截上述签到过程发送的数据包，可以看到有两次 `http` 请求：

1. `POST /api/user/checkin`

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j38rnxm8j31am10iqmn.jpg" alt="image.png" style="zoom: 50%;" />



2. `GET /api/user/status`

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j3b3rb60j31am0ts19u.jpg" alt="image.png" style="zoom: 50%;" />



#### 直接重放

两个包都丢进 `Repeater` 里面进行重放攻击分析。

可以看出，第一个请求是进行签到的，返回了签到的结果：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j3fj1tmwj31f00vg4li.jpg" alt="image.png" style="zoom: 50%;" />



第二个请求是获取当前会员信息的，主要包括注册邮箱、当前剩余天数等：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j3i414kdj31fa1a67wh.jpg" alt="image.png" style="zoom:50%;" />



#### 修改后重放

为了确定哪些 `header` 或者数据是有用的，我们可以把其中某个 `header` 或数据去掉进行重放，看返回结果是否正常，这里以第一个请求为例，第二个请求同理

##### 删除 Cookie

返回 `code` 为 `-2`，`message` 为 “没有权限”

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j3lxsbc0j31eq14kh8k.jpg" alt="image.png" style="zoom:50%;" />



##### 删除 token

返回 `code` 为 `1 `， `message` 为 `oops, token error`

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h7j3qr34jtj31eq1aqqts.jpg" alt="image.png" style="zoom: 50%;" />



##### 删除 Authorization

好像没啥影响......



## 签到结论

* 在签到请求中，必须要添加的 `header` 是 `Cookie` ，必须要添加的数据是 `token`
* 在查询请求中，必须要添加的 `header` 是 `Cookie` 



## 其他

其实自动化签到脚本已经写好了，后续有时间再来介绍如何使用（划掉）

* 基于 GitHub Action 的签到：https://github.com/xiabee/glados-checkin
* 基于 AWS Lambda 的签到：https://github.com/xiabee/aws-glados-checkin
