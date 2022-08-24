---
title: Mac 环境配置(3)——配置代理
date: 2022-8-24 12:15:23
tags:
  - mac
  - brew
categories:
  - coding
abbrlink: mac-proxy
---



## 概述

本篇不介绍如何搭建 VPN，仅介绍如何让 `MacOS` 的控制台连接到代理服务器，代理其应用层流量。

* 代理池：[Glados](https://glados.rocks/console)

* 代理工具：[Clashx](https://github.com/yichengchen/clashX)

* 代理服务器：`127.0.0.1`
* 代理端口：`7890`

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5i8yys9jrj31f40w8471.jpg" alt="image.png" style="zoom:100%;" />



## 终端代理

点击图标，可以看到“复制终端代理命令”

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5i94dcjdpj30i60twwo6.jpg)



复制内容如下：

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```



### 设置代理

把上述命令复制粘贴到终端里就可以了......

但是作为懒狗，每次输入这么多命令怎么行——直接修改 `~/.zshrc`（没装 `zsh` 的话可以修改 `~/.bash_profile`），在文件末添加以下内容，设置快捷命令：

```bash
function proxy() {
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=http://127.0.0.1:7890
    export all_proxy=socks5://127.0.0.1:7890
    echo -e "Proxy Server is running on 127.0.0.1:7890"
}

function unproxy(){
    unset http_proxy https_proxy all_proxy
    echo -e "Proxy is unset"
}
# 设置两个函数是为了同时方便“开”和“关”，不需要关代理的话只写"proxy"函数即可
```

修改完成后，重载配置文件即可：`source ~/.zshrc`

效果如下：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5i9azt6r7j30km08ydk6.jpg)



### 开启代理

```bash
proxy
# 开启终端代理
curl -I https://www.google.com -k
# 访问谷歌
```

返回 `200` 则说明访问成功，代理有效：

```bash
HTTP/1.1 200 Connection established

HTTP/2 200
content-type: text/html; charset=ISO-8859-1
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Wed, 24 Aug 2022 15:11:26 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Wed, 24 Aug 2022 15:11:26 GMT
cache-control: private
set-cookie: 1P_JAR=2022-08-24-15; expires=Fri, 23-Sep-2022 15:11:26 GMT; path=/; domain=.google.com; Secure
set-cookie: AEC=AakniGNoleB5QDOweRF8cAPBYO0XgdWWscKIH6sfpYd2rCtWCXcktNLnNBo; expires=Mon, 20-Feb-2023 15:11:26 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: NID=511=aPwxMWt8AQkAUPuU_WaVeDB8yxcXJ7asmgq2AA2JxFYPjZPoIkafjkR-FtpwH7VS-FJvJIApeW3wbKPEMTWn2vZXRJlAj3uPJF4KI_DpollKV66qceyhpm3gxQUgc7zjlc48H7m9FPZA_9rwd8HE67C9WQfr1VyF81iIINtW-S4; expires=Thu, 23-Feb-2023 15:11:26 GMT; path=/; domain=.google.com; HttpOnly
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000,h3-Q050=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,quic=":443"; ma=2592000; v="46,43"
```



### 关闭代理

```bash
unproxy
# 关闭代理
curl -I https://www.google.com -k
# 再次访问谷歌，很慢甚至直接超时
```

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5i9hs5p62j310m07sjz8.jpg)



### 测试说明

* Q：为什么不用 `ping` 来做测试
* A：注意，这里用到的 `clashX` 仅代理了 应用层 `http` 协议和传输层 `socks5` 协议的流量；而 `MacOS` 原生的 `ping` 命令使用的是 `ICMP` 协议，属于网络层，没有经过传输层和应用层的代理，所以直接在命令行里面 `ping` 是不能测试代理是否成功的。
* 所以使用 `curl -I` 来进行应用层的测试。



## 浏览器代理

推荐使用 [Proxy SwitchOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)，直接设置相应情景即可：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5i92owlffj31to0ww7e2.jpg)