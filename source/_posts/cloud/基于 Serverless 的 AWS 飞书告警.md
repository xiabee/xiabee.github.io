---
title: 基于 Serverless 的飞书告警系统（1）
date: 2023-7-7 12:00:00
tags:
  - cloud
  - lark
  - lambda
  - sns
categories:
  - cloud
abbrlink: aws-sns-lark
---

<meta name="referrer" content="no-referrer">


# 背景

贵司的告警系统迁移，再加上一些新增需求，我需要做一个能从 AWS 获取告警信息并发送给飞书（Lark）客户端的系统。

本篇主要介绍如何通过 Lambda 代码进行 AWS [SNS](#sns)（Simple Notification Service）消息转发，应用场景主要为已有 AWS 消息，但需要集成至第三方平台。



# 架构

* 通过 AWS 相关组件，把需要发送的消息发送给 AWS Lambda（无主机函数），再通过 Lambda 程序把消息发送给飞书机器人（Lark bot）

* 如图所示，详细代码见仓库：[AWS-Lark-Bot](https://github.com/xiabee/AWS-Lark-Bot)

![image-20230707143754035](https://s3.xiabee.cn/pic/2023/07/79327e8a698b68e02a90af1f12c015bb14c1e29ae85d6e0a492812c84abb10c1.png)



## 名词解释

### CloudWatch

AWS CloudWatch 是一个由 Amazon Web Services 提供的监控服务，用于监控 AWS 资源和运行在 AWS 上的应用程序。你可以使用 Amazon CloudWatch 收集和跟踪指标，这些指标表示了您的 AWS 资源和在 AWS 上运行的应用程序的性能。

以下是 CloudWatch 主要功能的概述：

1. **指标收集**：CloudWatch 可以收集诸如 CPU 使用率、数据传输速率、磁盘使用量等基本指标，并将其可视化。
2. **日志管理**：CloudWatch Logs 可以收集、存储、访问和监视应用程序和系统日志文件。这样可以使你更好地了解系统和应用程序的行为。
3. **事件管理**：CloudWatch Events 帮助你响应在 AWS 环境中更改的状态。例如，如果 EC2 实例的状态发生变化（如从运行状态变为停止状态），你可以配置 CloudWatch Events 在这种情况发生时发送通知。
4. **告警**：你可以设置 CloudWatch 告警，当某个指标超过预设的阈值时，CloudWatch 将发送一条消息或自动进行一些操作，如停止或启动一个实例。
5. **仪表板**：你可以在 CloudWatch 仪表板上创建可视化的图表和数据小部件，以直观地查看和分析 AWS 资源和应用程序的性能。

CloudWatch 是一个强大的监控工具，无论是对于 AWS 资源的监控还是对运行在 AWS 上的应用程序的监控都十分重要，能帮助你及时发现并处理问题，优化应用性能，同时满足一些特定的业务需求，如合规性报告等。

### SNS

<a name="sns"></a>

Amazon Simple Notification Service (SNS) 是 Amazon Web Services (AWS) 提供的一个完全托管的发布/订阅（Pub/Sub）消息服务，它使得系统和用户之间，或者系统和系统之间的消息传递变得容易且可靠。

以下是一些 SNS 的关键特性：

1. **发布/订阅模型**: 你可以创建一个或多个主题，然后允许应用程序和用户订阅这些主题以接收消息。当你向主题发布消息时，所有订阅的应用程序和用户都会接收到消息。
2. **多种通知类型**: SNS 支持多种类型的消息传递方式，包括 HTTP/HTTPS、Email、Email-JSON、Amazon SQS、AWS Lambda 和 SMS。
3. **批量消息传递**: SNS 支持向多个订阅者同时发送消息，这对于实现发布-订阅（Pub/Sub）模式的应用程序非常有用。
4. **可靠的消息传递**: SNS 提供了多种策略来确保消息的可靠传递，包括重试、死信队列等。
5. **持久性**: SNS 主题能够持久化你发布的消息，以便订阅者能够接收和处理。
6. **访问控制**: 你可以使用 AWS Identity and Access Management (IAM) 来控制谁可以发布消息，谁可以订阅主题，以及谁可以接收消息。

使用 SNS，你可以解耦微服务、分布式系统和无服务器应用，让每个组件可以独立运行和演进。同时，它可以用于发送各种类型的通知，包括系统警告、价格变动通知、订单状态更新等。



### Lambda

AWS Lambda 是一个无服务器计算服务，它可以运行你的代码并响应各种事件，如 HTTP 请求、数据库更新、队列服务消息、监控警告、文件上传等。你无需预置或管理任何服务器，Lambda 将自动为你的应用程序提供运行所需的容量，并且你只需为实际的计算时间付费。

以下是 AWS Lambda 的一些关键特性：

1. **事件驱动**: 你可以设置Lambda函数以响应各种事件源，如更改在Amazon S3存储桶中的文件、更新在DynamoDB表中的记录或自定义事件。
2. **扩展性**: 根据入站事件请求的数量，Lambda可以自动扩展你的应用程序。
3. **无服务器**: 你无需管理任何服务器，Lambda会自动运行你的代码。
4. **微秒级计费**: 你只需为实际的计算时间付费。当你的代码未运行时，没有任何费用。
5. **集成**: AWS Lambda可以与AWS的其他服务无缝集成，包括Amazon S3, Amazon DynamoDB, Amazon SQS, Amazon SNS, Amazon CloudWatch等。
6. **运行环境选择**: 你可以在多种编程语言中编写Lambda函数，包括Node.js, Python, Java, Ruby, C#, Go和PowerShell。

总的来说，AWS Lambda是一种高效、灵活的方式来处理你的服务和应用程序的工作负载。不管你正在构建微服务、实时文件处理和数据转换、web 应用，或者运行大数据工作负载，Lambda 都可以为你提供所需的计算资源。



# 基本交互

主要介绍一下

## 飞书机器人设置

### 创建机器人

可以直接在飞书群聊中创建：

![image-20230707145055526](https://s3.xiabee.cn/pic/2023/07/f7140bafa1d6aab6108a6bde373e660751dce1edb373512e5f30d5df3f15a26a.png)



### 查看 Webhook

注意管理这个 webhook，向此 url 发送特定格式的请求，可以直接使群机器人发送消息

![image-20230707145152575](https://s3.xiabee.cn/pic/2023/07/a5c779d87b97014aa795875fada7c1e85895729af7e39eec8777b242af8c0ce9.png)



## 飞书 Webhook 相关

消息体格式参考：https://open.feishu.cn/document/client-docs/bot-v3/add-custom-bot

以下为几个简单举例：保存刚刚的 webhook 地址，直接用 curl 进行测试

### 文本消息

```bash
exprot WEBHOOK=https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx
# webhook 替换为自己的
curl -X POST -H "Content-Type: application/json" \
    -d '{"msg_type":"text","content":{"text":"request example"}}' \
$WEBHOOK
```

此时提示发送成功：

![image-20230707145938031](https://s3.xiabee.cn/pic/2023/07/18a7621d9add3ba82499eebaf0f0d6dedb75b3aca9f7c604b3a52bd4b0a94596.png)

飞书客户端成功收到消息：

![image-20230707150007180](https://s3.xiabee.cn/pic/2023/07/bcb130c62b93dcea805aa2728ae63ce1fa2683f5d8ecbeff95cab61ce8773ec8.png)



### 卡片消息

消息结构体参考：https://open.feishu.cn/document/common-capabilities/message-card/message-cards-content/card-structure/card-content

```bash
exprot WEBHOOK=https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx
# webhook 替换为自己的
curl -X POST -H "Content-Type: application/json" \
  -d '{
    "msg_type": "interactive",
    "card": {
      "config": {
        "wide_screen_mode": true
      },
      "header": {
        "title": {
          "tag": "plain_text",
          "content": "卡片消息"
        },
        "template": "blue"
      },
      "elements": [
        {
          "tag": "div",
          "text": {
            "tag": "lark_md",
            "content": "\n ---\n 这是一条卡片消息"
          }
        },
        {
          "tag": "hr"
        }
      ]
    }
  }' \
  $WEBHOOK

```



发送成功：

![image-20230707150253860](https://s3.xiabee.cn/pic/2023/07/08dce2b3e1518a88b9c60064a80f8f3537cb5552c6ae7af74e4ee3baeff4ebff.png)



飞书客户端接收消息成功：

![image-20230707150328471](https://s3.xiabee.cn/pic/2023/07/4698650f2dd24400012141bb5c86086f95b5f632f6cbc23a5d79f5ee45a6c4c4.png)



# 代码简介

本篇介绍的 SNS 交互功能存放在 [SNS-Integration](https://github.com/xiabee/AWS-Lark-Bot/tree/main/SNS-Integration) 目录中。

## main.go

设置 webhook 地址，通过环境变量读取 webhook 最后数位哈希串

```go
// Lark bot webhook URL
secret := os.Getenv("WEBHOOK_KEY")
webhookURL := "https://open.feishu.cn/open-apis/bot/v2/hook/" + secret
```



读取 SNS 消息，并解析

```go
// get the SNS message
message := snsEvent.Records[0].SNS.Message
event, err := lib.ProcessJSON(message)
```



根据飞书卡片请求包格式，处理待发送的请求

```go
lib.ProcCard(event, &data)
payloadBytes, err := json.Marshal(data)
resp, err := http.Post(webhookURL, "application/json", strings.NewReader(string(payloadBytes)))
```



## lib/CardJson.go

飞书卡片消息结构体，用于解析卡片消息的 Json 结构。

## lib/SNSJson.go

SNS 消息结构体，用于解析 SNS 消息的 Json 结构。

## lib/Card.go

设置飞书卡片消息，设置卡片消息类型、卡片颜色、卡片内容（来自 `element.go`）等

```go
func ProcCard(event Event, data *CardMessage) {
	data.MsgType = "interactive"
	data.Card.Config.WideScreenMode = true
	data.Card.Header.Title.Tag = "markdown"
	data.Card.Header.Title.Content = event.Detail.Title
	data.Card.Header.Template = "blue"

	var element Element
	ProcElement(event, &element)
	data.Card.Elements = append(data.Card.Elements, element)
}
```



## lib/Element.go

处理 SNS 消息，写入飞书卡片消息结构体中



# 附录：Lambda 使用方法

（未完待续）

# Reference

> https://open.feishu.cn/document/client-docs/bot-v3/add-custom-bot
>
> https://github.com/xiabee/AWS-Lark-Bot/tree/main/SNS-Integration
