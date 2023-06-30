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

后来又推出了个人的 lite 版，[ossinsight-lite](https://github.com/pingcap/ossinsight-lite) ，有点酷炫，而且里面教了大家如何白嫖 tidbcloud 来搭建自己的 ossinsght。这里我就做个搬运工，顺便记录一下踩坑过程。

<a id="ex"></a>

![image-20230630165118505](https://s3.xiabee.cn/pic/2023/06/1f34f76818140376e23db746db88a7c32965c1ef8dc927bf07760de916844579.png)



## 准备工作

需要有一下三个服务的账户：

1. [Github](https://github.com) 账户：用于fork并运行您自己的数据流水线，以获取您在GitHub上的活动记录；
2. [TiDB Serverless](https://tidbcloud.com) 账户：免费的云数据库，用于免费存储个人数据；
3. [Vercel](https://vercel.com/) 账户：用于部署前端看板



# 创建云数据库

> Reference: https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/database.md

## 创建 Serverless 集群

进入 [tidbcloud](https://tidbcloud.com/console/clusters/create-cluster?utm_source=ossinsight&utm_medium=lite)，创建一个 Serverless 集群（富哥想创建付费集群当我没说）

![image-20230630220026636](https://s3.xiabee.cn/pic/2023/06/d07de8d0c9f1a04cd14c2b8f9427a3d0588183966960f00d74677a32f438cdd6.png)



等待几分钟，创建完成后大概长这样：

![image-20230630172831319](https://s3.xiabee.cn/pic/2023/06/fa24f9da8f43ba159fab0b220d839d416b469d10b0be69cc5fa0a04672461078.png)



## 记录登录信息

点击右上角 `Connect` 按钮，找到云数据库的端点、端口、用户名、密码等信息，这是 `root`用户的信息，之后如果按照官方文档操作应该使用这个账户就可以了。

![image-20230630173209367](https://s3.xiabee.cn/pic/2023/06/b5af5f6e5922cc518803ea408192c68f41f0f8acdde4d7cf88b0c8d49c628346.png)

但是如果你想创建一个独立的非 `root` 用户，请往下看。

## 创建用户注意事项

如果你不想想[官方文档](https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/database.md)一样使用 root 用户登录，而是创建一个专属的数据库用户的话，需要注意以下事项：

### 用户名前缀固定

TiDBCloud Serverless 共享存储卷，不通租户用户名前缀实现空间隔离，因此我们不能完全自定义用户名，需要有特定的用户名前缀。

![image-20230630221045803](https://s3.xiabee.cn/pic/2023/06/249756c2024d861161f4afeb4e78e858fa51c41be808e61b9de9e56951192d53.png)

> https://docs.pingcap.com/tidbcloud/select-cluster-tier#user-name-prefix



比如我的 root 用户名为`44Kz3Lonut9UUp2.root` （随便举例，非真实用户），我想创建一个 `xiabee` 用户，就应该创建为 `44Kz3Lonut9UUp2.xiabee`

```mysql
create user '44Kz3Lonut9UUp2.xiabee'@'%' identified by 'password';
```

![image-20230630221525926](https://s3.xiabee.cn/pic/2023/06/6d24a48049a4da2f010043f571d6f07cc53a6e8f56b8cbdccdaf5a777ac93010.png)

用户创建成功



<a id="pass-id"></a>

### 密码设置

如果你的 `GitHub Action` 中遇到以下报错：

```bash
/usr/local/lib/ruby/3.2.0/uri/rfc2396_parser.rb:176:in `split': bad URI(is not URI?): mysql2://xxxxxxxxxxxx.xiabee.github:###############@gateway01.ap-southeast-1.prod.aws.tidbcloud.com:4000/github_repos (URI::InvalidURIError)
```

那很可能是因为 TiDB 用户的密码中包含了特殊字符，如果遇到这种情况应该在 `SECRET` 填入密码时使用 `url` 编码；或者设置一个不包含特殊字符的密码。

详细内容参考这篇 [issue](https://github.com/pingcap/ossinsight-lite/issues/120)

![image-20230630222726938](https://s3.xiabee.cn/pic/2023/06/1aec9a83221eb9e3f2113eb42fd4030872dacb33e98193d77e5721d599d1594d.png)

<a id="grant-id"></a>

### 用户授权

最终在 `Github Action` 里面要创建两个库，一个是 `github_personal`，一个是 `github_repos`，我们登录的用户需要有这两个库的写权限。

```mysql
GRANT ALL ON github_personal.* TO '44Kz3Lonut9UUp2.xiabee'@'%';
GRANT ALL ON github_repos.* TO '44Kz3Lonut9UUp2.xiabee'@'%';
```

接下来应该可以顺利运行 `Github Action` 了



# 设置 Github Action

> Reference: https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/repo-and-action.md

设置 Action 分为三步：

1. [Fork ossinsight-lite repository](https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/repo-and-action.md#fork-ossinsight-lite-repository)
2. [Setup GitHub Action secrets](https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/repo-and-action.md#setup-github-action-secrets)
3. [Enable and run workflow](https://github.com/pingcap/ossinsight-lite/blob/main/docs/setup/repo-and-action.md#enable-and-run-workflow)



## Fork 仓库

![image-20230630225059011](https://s3.xiabee.cn/pic/2023/06/a6428ac47379a540857898b83caa058bbfc30fe90589beed60e43373bdab2dbc.png)



## 设置 Action Secrets

Settings :arrow_right: Secrets and variables :arrow_right: Action

![image-20230630225803899](https://s3.xiabee.cn/pic/2023/06/5b273f77b85d04e866557bb2b381674c15dbf6591983cee57d0d1e26be978b27.png)

设置一个名为 `DATABASE_URL` 的环境变量，其值为

```go
mysql://<user>:<password>@<host>:<port>
```

<details>
<summary>点击展开示例</summary>
  如果你的 TiDB 连接凭证信息如下：
<pre>
  host: 'gateway01.us-west-2.prod.aws.tidbcloud.com',
  port: 4000,
  user: '4Budszs5sxiUZ65.root',
  password: 'Us1h2MRraVB4zfpU',
  ssl_ca: /etc/ssl/cert.pem
</pre>
  那么 DATABASE_URL 的值应该设置为 
  <pre>
  mysql://4Budszs5sxiUZ65.root:Us1h2MRraVB4zfpU@gateway01.us-west-2.prod.aws.tidbcloud.com:4000
  </pre>
  </details>



## 开启 workflow

允许 workflow 运行

![WeChat1d9b20fbc06b9243273dc8fb51967a1e](https://s3.xiabee.cn/pic/2023/06/d3c48f9a44ce95eeaf7706e8e8b209355bd056cf8f78f0d6854795af4e25fa7a.jpg)



第一次开启时需要手动运行

![image-20230630230909496](https://s3.xiabee.cn/pic/2023/06/4bfb24abb7e7c3c6e3dc673069d1c54cbaf19cf653cd4cf588f9a977f31bbbb7.png)

## 问题处理

* 可以参考官方仓库的[文档](https://github.com/pingcap/ossinsight-lite/blob/main/README.md#troubleshooting)
* 大部分问题应该是我前面提到的 [密码设置](#pass-id) 和 [用户授权](#grant-id)



# 部署 Vecel

通过 [Vecel](https://vercel.com/dashboard) 部署我们的前端 dashboard

## 导入项目

进入 Vecel 控制台，创建项目

![image-20230630231733963](https://s3.xiabee.cn/pic/2023/06/2696d0bcefdfbe447f4463edcbcb2cfdd5026c8d28805e838712798b8a892fe1.png)



部署刚刚 Fork 的那个仓库：`ossinsight-lite`

![image-20230630232010050](https://s3.xiabee.cn/pic/2023/06/694bcdcf68b64f3509e9adb57f28ab915b0f7acba6fcebdbac4eea615ad06c00.png)



直接 Deploy，项目会在几秒内失败，因为我们还没有配置凭证信息。但是没有关系，继续往后走

![image-20230630232141642](https://s3.xiabee.cn/pic/2023/06/53f76fd0e02888f7ac65a4ce1a8f637b7ccb2227bd9d9bce28bc7e0301a4fc0d.png)



## 创建 TiDBCloud 集成

进入 [TiDBCloud 集成主页](https://vercel.com/integrations/tidb-cloud)，选择项目之后一路 continue 即可

![image-20230630232414840](https://s3.xiabee.cn/pic/2023/06/86a819edb4632fbe2e4eda5a3afe41c27103722e5afb082a8f5b7119cfe13836.png)

集成后如下：

![image-20230630232715305](https://s3.xiabee.cn/pic/2023/06/cf2c7c59035fb4772033eca6f895dd829282d949978cedc870376e5f047753bd.png)



## 重新部署 Vecel

![image-20230630232804146](https://s3.xiabee.cn/pic/2023/06/df42e543761848c9159217bfae7b65dc9efe3eaa1dd54935dd28086dbf439465.png)

等待几分钟，待其运行完毕。

![image-20230630233238615](https://s3.xiabee.cn/pic/2023/06/14af16ea3b39ba78a37dcf1d928c7a48f7765d519ed60b78aaa0e9373a410c08.png)



## 设置自定义域名

![image-20230630233431061](https://s3.xiabee.cn/pic/2023/06/84eb2520771b39d5fdb501ec5852e2d2ecc2195c4e0bf8074351b39dc5408542.png)



# 最终效果

* 完整效果如 [背景](#ex) 所示
* 当然也可以直接引用进 markdown 里面



![image-20230630233505413](https://s3.xiabee.cn/pic/2023/06/e4ba111802304cc9917d91db59f8d4a3b777aaa02e7411e53f8edc6c41badfee.png)



# 参考文档

> https://github.com/pingcap/ossinsight-lite
