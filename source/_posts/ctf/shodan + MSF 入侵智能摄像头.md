---
title: Shodan + MSF 入侵智能摄像头
date: 2021-3-10 20:00:23
tags:
  - ctf
categories:
  - ctf
abbrlink: shodan-cam
---



## 0xFF 免责声明

由于传播、利用此文提供的信息而造成的任何直接或间接的后果，均有使用者承担，本站不为此承担任何责任。



## 0x00 简介 

[shodan](https://www.shodan.io/) 是一个比较 dangerous 的搜索引擎，用于查看联网的智能设备——通常作为渗透测试的信息收集工具； 

[MSF](https://www.metasploit.com/)，Metasploit-Framework，一款开源的渗透工具，用于检测漏洞、生成 payload、制造后门文件等。 

>  Shodan 是一个开放式搜索引擎，它专注于搜索互联网上的设备，而不是网页。Shodan 可以搜索各种类型的设备，包括服务器、路由器、摄像头、智能家居设备等。
>
> Shodan 提供了一个简单的搜索界面，您可以输入关键字查询，Shodan 会返回匹配的设备列表。您可以通过 Shodan 搜索设备的位置、操作系统、IP 地址、开放的端口等信息。
>
> Shodan 可以用于安全研究和测试，它可以帮助您识别暴露在互联网上的设备，以便您可以评估这些设备的安全性。此外，Shodan 还可以帮助您识别潜在的漏洞和安全问题，并提供相关的建议和修复方案。
>
> 请注意，使用 Shodan 可能存在安全风险，因为您可能会暴露自己的私人信息或敏感信息。因此，在使用 Shodan 之前，请仔细考虑您的目的和风险，并遵循所有相关的法律和道德规范。

![shodan主页 ](https://s3.xiabee.cn/pic/2023/03/514f0d8b43aa77dd10b70660c9acf77dfa4cb8e5a0fda741a68207843925c38c.png)



<img src="https://s3.xiabee.cn/pic/2023/03/c55da476153d89febc88d4d58cb83fa83353099960285e6443836a3b2ecf4d25.png" alt="MSF控制台" style="zoom:67%;" />

 

## 0x01 Shodan直接测试 

* 注册账号 
* 搜索框直接搜索 
* 进入查看 

例如直接搜索默认密码：

![image-20230309203608224](https://s3.xiabee.cn/pic/2023/03/b9e1551b850015c423ee0e35ab209881500ada97377adc82a51885fbfca8ef03.png)



搜索结果：

![image-20230309203623351](https://s3.xiabee.cn/pic/2023/03/248e35c4b09cda750173b59ec6fb77a3e69131b15a69628b4a97b5619fb35da1.png)



## 0x02 结合 MSF 测试 

### 启动 console

* 进入 msfconsole 
* 搜索 shodan 相关模块 
* 选择模块 

```bash
msfconsole 
search shodan 
use auxiliary/gather/shodan_search 
```

![img](https://s3.xiabee.cn/pic/2023/03/655588e98f4a93aa737b37e29e72167f857424683653cc9c8f28818eb1726921.png)

切换模块



### 查看选项 

```bash
show options
```

 ![img](https://s3.xiabee.cn/pic/2023/03/dde521030fa35d70a58ba6ea08635605405c53d7c1dea15af2b844bc95ebba6b.png)

查看options 



必须填写的参数是 `SHODAN_APIKEY` 和 `QUERY`，`APIKEY` 在 `shodan` 官网注册即可获得，`QUERY` 为查询关键字，自行设置即可。

![img](https://s3.xiabee.cn/pic/2023/03/04e01a43ba09753feeff0a7f62b147c0cecfb996b9e59d71798b50c485452dac.png)

### 设置参数 

```bash
set ShODAN_APIKEY xxxx
# 直接复制刚才的APIKEY
set QUERY netcam
# 这里以网络摄像头为例，也可以设置为其他的
```

![img](https://s3.xiabee.cn/pic/2023/03/5bbfe267a7c4b11e4eeb72cd745d14dacb21dd8642362181eadab8f2b89d251c.png)



### Exploit

![img](https://s3.xiabee.cn/pic/2023/03/1b860ddda53df29bae1f6ded4dec5c1884e975102d513c254f29b499fae014dd.png)

MSF运行结果



## 0x03 渗透 

由于本次行动没有任何授权，我们点到为止，随机测一个弱口令（x 

随便抓了一个站点，浏览器访问：  

![img](https://s3.xiabee.cn/pic/2023/03/c3a8090986a3fa69c9dd468b776db80e2a67adc783685c2d58d03db2f72877fc.png)



网络监控管理界面 默认密码跑一遍，秒出结果——这家伙居然没设密码...... 

直接Login点进去： 

![img](https://s3.xiabee.cn/pic/2023/03/d6742285853c8d78ca67b3f9a6413d357cdaf0a0196a31c2a9fb7da93751581d.png)

 管理员界面 懒癌晚期，不想装Flash了，就没继续了(x) 



## 0x04 小结 

* 作为针对智能设备的搜索引擎，是一个很好的信息收集工具；
* 但为渗透测试提供便利的同时，也为真正的攻击者创造了工具，使针对智能设备的攻击更加容易； 
* 简单点，生活的方式简单点，不需要联网的设备就别联网...... 
* 默认密码记得改一下啊铁汁们