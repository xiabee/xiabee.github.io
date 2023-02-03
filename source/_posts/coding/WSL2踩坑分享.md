---
title: WSL2踩坑分享
date: 2020-10-26 20:00:23
tags:
  - wsl
  - linux
categories:
  - coding
abbrlink: wsl2-about
---



WSL：[Windows Subsystem For Linux](https://docs.microsoft.com/zh-cn/windows/wsl/about)

前段时间听说WSL2很香，然后试了一下，确实很香......如果你不是一个WEB狗

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypdg26yuj30zk0detc4.jpg)

WSL2的安装我就不具体写了，太简单了，直接看[官网教程](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)



## 0x00 与WSL的区别

WSL1大家应该都很熟悉，WSL2也问世挺久了，我就不详细嗦了，直接看看官网解释的区别：

![](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypdtko92j30wk0kw7a0.jpg)

摘自[微软官网](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions)



整体上看......这俩完全就不是一个东西嘛：`wsl`只是一个单纯的`shell`，`wsl2`简直就是一台`虚拟机`

`WSL2`从`OS`层实现了独立，不再与宿主机共享`OS`，解决了许多`二进制`狗的蜜汁BUG，但是这种独立性给`WEB狗`创造了更多的烦恼



## 0x01 主要问题

### IPV6

没有`IPV6`，失去了免流的梦想......

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvyphpiu4dj30jn03r3zk.jpg)

微软，不愧是您



### 服务不通

最主要的问题还是网络层与宿主机不互通：

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypfg8bqcj30tx0lgwt7.jpg)

微软[官方说明](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions)



`ifconfig`一下你会惊奇的发现它没有`192`网段地址：

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypg1dsv8j30s10f9k5l.jpg)

ifconfig



再来看看 **WSL2** “独一无二的IP地址的**虚拟化以太网适配器**”：

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypgcb4pwj30l8097771.jpg)

图源：https://blog.csdn.net/swordsm/article/details/107948497



果然很复杂。再看一眼昔日称霸一方的**WSL1**的网络拓扑：

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypgpl41ij30ao06a74n.jpg)

图源：https://blog.csdn.net/swordsm/article/details/107948497





两图对比，发现WSL1使用的是宿主机的网络，可以直接理解为**一台机器**，在配置上过于简单，但正因为网络共享，很多应用不能在windows和Linux中同时运行（比如Windows的phpstudy和Linux的docker）

WSL2网络层使用了独立的虚拟网卡，与宿主机进行**桥接**，网络上实现了独立，但是也造成了麻烦，比如本地服务不互通：

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypgxilywj30e10evmz7.jpg" alt="image.png" style="zoom:80%;" />

比如内网请求被防火墙干掉：

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvyphb084ej30nq05jn0r.jpg)

wsl2 ping 宿主机被拦截



![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvyphgn80dj30jx0avqa9.jpg)

宿主机 ping wsl2 能 ping 通



可能是WD把这个网桥干掉了吧......



### Docker

Docker  Docker  Docker!

也可能是我发行版的问题，在运行docker的时候一直报错，一直启动不了：

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypig4e3vj30sh0e2dv6.jpg)

```bash
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination`, error: exit status 4
INFO[2020-10-26T18:26:49.534239700+08:00] stopping event stream following graceful shutdown  error="<nil>" module=libcontainerd namespace=moby
INFO[2020-10-26T18:26:49.534502600+08:00] stopping event stream following graceful shutdown  error="context canceled" module=libcontainerd namespace=plugins.moby
INFO[2020-10-26T18:26:49.534542500+08:00] stopping healthcheck following graceful shutdown  module=libcontainerd
failed to start daemon: Error initializing network controller: error obtaining controller instance: failed to create NAT chain DOCKER: iptables failed: iptables --wait -t nat -N DOCKER: iptables v1.8.5 (nf_tables):  CHAIN_ADD failed (No such file or directory): chain PREROUTING
 (exit status 4)
```

启动很失败（x）





## 0x02 解决方案

### IPV6：无解

`IPV6`的问题没法解决，干脆忘了它吧，等微软开发下一代产品



### 服务不通：手动设置+localhost

Linux请求被拦截的问题......关闭专用网络防火墙可以解决：（确实是WD把这个网桥干掉了）

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypit0y20j30f60b8aax.jpg)

关闭专用网络防火墙





![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypj005mej30u70c5naa.jpg)然后终于ping通了



网络问题大致有两种解决方案：直接**localhost**或者手动配置**端口转发**

**localhost**大概是微软留下的最后的希望，在不能使用`127.0.0.1`的情况下，**localhost**可以直接访问本地服务

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypj5h2z2j313u0nbtuu.jpg)

这里直接跑了一个DVWA



端口转发可以直接参考官方文档，在宿主机的Powershell中配置：

```bash
netsh interface portproxy show all
# 查看转发

netsh interface portproxy add v4tov4 listenport=4000 listenaddress=0.0.0.0 connectport=4000 connectaddress=192.168.101.100
# 添加转发

netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=8000
# 删除转发
```



### Docker：强制初始化

主要错误原因：

```bash
 iptables v1.8.5 (nf_tables):  CHAIN_ADD failed (No such file or directory): chain PREROUTING
```

> `docker.io`用`iptables`初始化NAT网络，而`Debian buster`使用 `nftables` 而不是 `iptables`，导致`dockerd`不能正常完成NAT初始化，出错退出。



那么正确的解法应该就是强制使用iptables进行初始化：

```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

# 重启docker
sudo service docker restart
```

然后就能跑了......



![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvypjkvcqoj316o04bn27.jpg)



## 最后

其他的BUG可能忘记了，下次想起来再写





## Refference

> https://blog.csdn.net/dqwjack/article/details/107699985 
>
> https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions 
>
> https://blog.csdn.net/swordsm/article/details/107948497 
>
> 
>
> Reference
