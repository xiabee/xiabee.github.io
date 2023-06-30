---
title: PicGO 连接 AWS S3
date: 2023-4-19 19:30:23
tags:
  - cloud
  - aws
categories:
  - cloud
abbrlink: pichost-picgo
---



## 简介

[PicGo](https://github.com/Molunerfinn/PicGo):  一个用于快速上传图片并获取图片 URL 链接的工具，可以直接使用命令行上传，也可以用 GUI 上传，使用和二次开发都很方便

在上期 [S3 图床 ](https://blog.xiabee.cn/posts/aws-pichost/)中我们提到了 PicGO，这里我们简单介绍一下如何配置 PicGO 上传图片到 S3。



## PicGO 设置

### 下载运行

GitHub 里面直接找到最新 [release](https://github.com/Molunerfinn/PicGo/releases/) 的稳定版本，下载安装即可。

如果你是比较新的 macOS，可能会遇到 “已损坏，无法打开” 的问题，可以参考[之前的博客](https://blog.xiabee.cn/posts/app-error/)

正常安装成功后，通过命令行启动，或者直接 `command + space` 聚焦搜索启动：

![image-20230419185608599](https://s3.xiabee.cn/pic/2023/04/a16efad700ad949e52a8d28286cca122585292fe09d87659163b8703cb438df3.png)



### 插件安装

打开主界面，插件设置中搜索`amazon s3 uploader`，安装相关插件

![image-20230419185745234](https://s3.xiabee.cn/pic/2023/04/a2484faeb2c392ccd539363e2157e52f04d46a4f466b6765ebec24f16691d2c5.png)



### 图床设置

图床设置 ➡️ Amazon S3

![image-20230419190247142](https://s3.xiabee.cn/pic/2023/04/949e427d51c3eb2959bfed3ba9e95b33387ac737b9d8b87114133380bcf0ce93.png)

有几个要注意到地方：

* 应用密钥 ID：需要在 AWS IAM 中创建一个账户，并为该账户创建访问密钥，该密钥的 ID（即 AK/SK 中的 AK） 填写到这里；
* 应用密钥：刚刚创建的密钥本体（即 AK/SK 中的 SK）填写到这里；
* 桶：AWS S3 中创建的 bucket，注意这里填写的是 bucket name，不是 bucket arn；
* 自定义域名：最后会生成的图片域名，和图片上传位置没有直接关系，一般填自己的 CDN 即可



注意：

* 应用密钥（SK）不要暴露给别人！
* 应用密钥（SK）不要暴露给别人！
* 应用密钥（SK）不要暴露给别人！

重要的话说三遍（不是）

* AK/SK 密钥对是第三方程序调用 AWS 接口的常见凭据，而且失效时间难以控制，一般都是长期有效，所以一旦泄漏可能会直接导致大门敞开（x）

* SK 一般要配合 AK 一起使用，但 AK 有固定的格式，如果 SK 泄漏，攻击者可以通过构造大量 AK 的方式去碰撞密钥对（如果 AK/SK 一起泄漏就更离谱了...）



## AWS IAM 设置

刚刚提到了 AK / SK，如果你还没有创建的话，可以参考一下我的权限分配。

### 登录 console

命令行的教学以后再来写，这里先教大家怎么用网页来操作（x）

直接搜索栏搜索 IAM

![image-20230419191411855](https://s3.xiabee.cn/pic/2023/04/935d9f78e2c89246ec08055a5fe296179808ed9efde42eae7d4b256ded36ffd9.png)



### 创建 user

访问管理 ➡️ 用户 ➡️ 添加用户

![image-20230419191500947](https://s3.xiabee.cn/pic/2023/04/eb652f42611aeff24045310ad802566e7dda43a41bc5ded885165535f80f2599.png)



创建的时候可以什么权限都不给，我们后面再来详细设置。



### 创建内联策略

找到刚刚创建的用户 ➡️ 权限 ➡️ 添加权限 ➡️ 创建内联策略

![image-20230419191748756](https://s3.xiabee.cn/pic/2023/04/393bd202a39e783836d7dd85e0d45d1188747b479156be1c7a6fc4293165987f.png)



不熟悉的小伙伴可以通过可视化编辑器搜索 “S3” 相关写权限：

![image-20230419191940777](https://s3.xiabee.cn/pic/2023/04/76790a0233391e08834c8f96e8968861eab0308242c72e2b7d57b18ff1b01b04.png)



图片上传只需 put 相关权限，但是方便调试，我给了 list 部分权限。

同时可以通过“资源”模块限制用户（或者说是他的 AK/SK）可以访问资源范围，比如我限制只允许访问我图床的 bucket 中特定目录，其他资源不可访问：

![image-20230419192251992](https://s3.xiabee.cn/pic/2023/04/9789e6e952e8ba080e4f95bc4f377bdb35bb18563ab36207a2dca52140b480f5.png)



总体编辑一下大概如上图所示，会生成对应的权限 JSON，如果是对 AWS IAM 比较熟悉的小伙伴，可以直接编写 JSON：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:ListBucketMultipartUploads",
                "s3:AbortMultipartUpload",
                "s3:ListBucket",
                "s3:InitiateReplication",
                "s3:PutBucketCORS"
            ],
            "Resource": [
                "arn:aws:s3:::xiabee-storage-host/pic/*",
                "arn:aws:s3:::xiabee-storage-host"
            ]
        }
    ]
}
```

这里严格限制密钥权限，满足最小特权原则，即使密钥泄漏了也不能对我们 AWS 账号其他的资源产生影响。



### 创建访问密钥

为刚刚的用户创建访问密钥：安全凭证 ➡️ 访问密钥 ➡️ 创建访问密钥

密钥和用户强绑定，访问密钥的权限为刚刚设置的用户的权限

![image-20230419192652957](https://s3.xiabee.cn/pic/2023/04/7ba42ac6af470cab7619183a103014ee597da9fac3682d331864b566d8f435d2.png)



然后把密钥信息填写到 PicGO 里面即可：

![image-20230419190247142](https://s3.xiabee.cn/pic/2023/04/949e427d51c3eb2959bfed3ba9e95b33387ac737b9d8b87114133380bcf0ce93.png)



## 测试连接

此时随便上传一张照片看看：

![image-20230419193116231](https://s3.xiabee.cn/pic/2023/04/00cc21e3e3ec20ef2bbe820247221e32430f17e37758b7605cce501bb502358e.png)



在存储桶中已经找到该图片：此时说明连接成功

![image-20230419193357278](https://s3.xiabee.cn/pic/2023/04/0142bed68384b4818a4c37cc357f42ee68874ca48d5b35fe628040a76ff04645.png)



## Typora 设置

每次写博客都要手动传图肯定是不行的，非常累（懒癌晚期）

这里我们用 Typora 自带的传图功能：Typora ➡️ 设置 ➡️ 图像 ➡️ 插图片时...

![image-20230419193656967](https://s3.xiabee.cn/pic/2023/04/243fac296cb761afc78952a1e49823b5a6fdb946159093f396db2b05bfaba9f2.png)



验证一下：

![image-20230419193808498](https://s3.xiabee.cn/pic/2023/04/9d2899bfbb621390086e943e0b178c4f943fe46f7c0ae0f8779568122ffa882a.png)

注意这个每一次验证都会上传一次 logo（两张小图片），点多了的话记得去 bucket 里面删掉......bucket 里面放垃圾也是要收费的（x）
