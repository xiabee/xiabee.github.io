---
title: 10 分钟内搭建自己的 Oss-Insight-lite
date: 2023-6-30 12:00:23
tags:
  - cloud
categories:
  - cloud
abbrlink: ossinsight-lite
---

<meta name="referrer" content="no-referrer">



# 背景

之前[贵司](https://www.pingcap.com/?from=en)做了一个 [OSS Insight](https://ossinsight.io/) ，是一个针对开源软件项目的可视化和分析平台。它提供了对开源项目的全面分析和可视化展示，帮助用户了解项目的活跃度、贡献者、代码质量、依赖关系等信息。

后来又推出了个人的 lite 版，[ossinsight-lite](https://github.com/pingcap/ossinsight-lite)，里面教大家如何白嫖 tidbcloud 来搭建自己的 ossinsght。这里我就做个搬运工，顺便记录一下踩坑过程。

![image-20230630165118505](https://s3.xiabee.cn/pic/2023/06/1f34f76818140376e23db746db88a7c32965c1ef8dc927bf07760de916844579.png)



# 准备工作

需要有一下三个服务的账户：

1. [Github](https://github.com) 账户：用于fork并运行您自己的数据流水线，以获取您在GitHub上的活动记录；
2. [TiDB Serverless](https://tidbcloud.com) 账户：免费的云数据库，用于免费存储个人数据；
3. [Vercel](https://vercel.com/) 账户：用于部署前端看板



# 部署过程

## 创建云数据库

### 创建 Serverless 集群

进入 [tidbcloud](https://tidbcloud.com/console/clusters/create-cluster?utm_source=ossinsight&utm_medium=lite)，创建一个 Serverless 集群（富哥想创建付费集群当我没说）

![image-20230630172831319](https://s3.xiabee.cn/pic/2023/06/fa24f9da8f43ba159fab0b220d839d416b469d10b0be69cc5fa0a04672461078.png)



### 记录登录信息

点击右上角 `Connect` 按钮，找到云数据库的端点、端口、用户名、密码等信息

![image-20230630173209367](https://s3.xiabee.cn/pic/2023/06/b5af5f6e5922cc518803ea408192c68f41f0f8acdde4d7cf88b0c8d49c628346.png)



### 创建用户注意事项

如果你不想想[官方文档](https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/database.md)一样使用 root 用户登录，而是创建一个专属的数据库用户的话，需要注意以下事项：

#### 用户名前缀固定

TiDBCloud Serverless 共享存储卷，不通租户用户名前缀实现空间隔离，因此我们不能完全自定义用户名，需要有特定的用户名前缀。

比如我的 root 用户名为`44Kz3Lonut9UUp2.root`，我想创建一个 `xiabee` 用户，就应该创建为 `44Kz3Lonut9UUp2.xiabee`

```bash
create user '44Kz3Lonut9UUp2.xiabee'@'%' identified by 'password';
```



(未完待续)

# 参考文档

> https://github.com/pingcap/ossinsight-lite
