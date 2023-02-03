---
title: GitHub配置SSH Key
tags:
  - git
  - ssh
categories:
  - coding
abbrlink: github-ssh
date: 2020-08-27 19:56:35
---

在Github上提交代码，每次 `push` 都需要输入一次密码，特别麻烦，所以现在就记录一下用`SSH`登陆GitHub的过程

先来康康这两个登陆方式有啥区别：

```bash
https://github.com/xiabee/fucking-views.git
# use https
git@github.com:xiabee/fucking-views.git
# use ssh
```

第一种直接使用https，可以直接浏览器访问，方便查阅，但是每次`push`都要输入密码

第二种使用ssh，不能直接访问，但是在`push`的时候可以保留密钥文件

个人感觉，在开发的时候，还是使用ssh比较方便。

那就康康如何配置SSH：



## 0x00 配置全局信息

仅第一次使用的时候需要配置，如果已经配置过了直接忽略

查看配置信息：

```bash
 git config --list
```

添加全局变量：用户名、邮箱

```bash
git config --global user.name "xiabee"
git config --global user.email  "xiabee@foxmail.com"
```



## 0x01 生成SSH密钥对

先检查一下本地是否存在密钥对

```bash
cd ~/.ssh
ls
```

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsyrgz0a2j30a704bdh9.jpg)

`id_rsa` 和 `id_rsa.pub` 分别是ssh的私钥和公钥 



如果没有的话就现场生成一个：

```bash
ssh-keygen -t rsa -C "xiabee@foxmail.com"
# 邮箱填自己的
```



![注释记得去掉...](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsyrxg1n7j30hi06twj1.jpg)



## 0x03 配置密钥对

查看公钥：

```bash
cat id_rsa.pub
# 查看私钥信息
```

![公钥信息](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsyshwq6hj30b8054q6f.jpg)

配置GitHub公钥信息：

### settings：

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsyt0t91kj30790h1ta5.jpg" alt="img"  />

### SSH and GPG keys

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsyte6ds2j30fu0ge424.jpg" alt="img"  />

### new SSH key

把公钥拷贝进去即可

<img src="https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsytx02pqj30qj0flq51.jpg" alt="img" style="zoom: 50%;" />

## 0x04 验证

```
$ ssh -T git@github.com
Enter passphrase for key '/c/Users/14793/.ssh/id_rsa':
Hi xiabee! You've successfully authenticated, but GitHub does not provide shell access.
```

![img](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvsyumdnt6j30k4031abm.jpg)

出现这个就可以使用`ssh`了



## 0x05 如果已经 git clone 了 https 怎么办

进入 `/.git` 文件夹，找到 `config` 文件，修改 `url` 的值即可

```bash
[remote "origin"]
	url = git@github.com:xiabee/fucking-views.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

