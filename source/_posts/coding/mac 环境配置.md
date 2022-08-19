---
title: Mac 环境配置(1)——完美终端
date: 2022-8-16 12:00:23
tags:
  - mac
  - brew
categories:
  - coding
abbrlink: mac-init
---



## 环境说明

公司发了新电脑，`M1 Pro` 的 `MacBook Pro`，用了几天体验还不错，这里介绍一下怎么在 `Mac` 里打造一个能让颜狗落泪的漂亮终端（划掉）

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h58pm1gs0uj30y20iu0za.jpg" alt="image.png" style="zoom:100%;" />



## “完美”标准

* 我是颜狗，终端好看是第一标准
* 自动补全应该是一个终端的必备需求
* 最好有语法高亮
* 最好能通过键盘唤醒

大概长这样：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cfxyz9ayj310y0pgqn1.jpg" alt="image.png" style="zoom:100%;" />



## 准备工作

需要安装一些软件 / 字体等。

### 字体

后面会用到 `p10k` 相关字体，前往 https://github.com/romkatv/powerlevel10k#manual-font-installation 下载并安装 `MesloLGS NF Regular.ttf` 即可。

### iTerm2

#### 下载与安装

一个终端工具，新款的 Mac 应该有预装，如果没有的话可以去[官网](https://iterm2.com/)下载安装一个。

在 `Application` 目录下，找不到的话也可以 `command + [space]` 进行查找：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cg5x52ttj311i05cjs7.jpg" alt="image.png" style="zoom:100%;" />



#### 完全磁盘访问权限

>  注：建议为 iTerm2 打开完全磁盘访问权限，避免出现默认 Terminal 能够执行正确，iTerm2 因为权限问题导致执行有误的情况出现。

* 左上角苹果图标 ➡️ 系统偏好设置

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cgen3sifj30ds0hcmzx.jpg" alt="image.png" style="zoom:100%;" />

* 系统偏好设置 ➡️ 安全性与隐私 ➡️ 隐私 ➡️ 完全磁盘访问权限 ➡️ iTerm

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cgbwvu3gj31140wm7df.jpg" alt="image.png" style="zoom:100%;" />



### zsh

Mac 系统默认安装了 `zsh` ，用以下命令把默认 `Shell` 改成 `zsh` 就行：

```bash
chsh -s /bin/zsh
```



### oh-my-zsh

> Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration. It comes bundled with thousands of helpful functions, helpers, plugins, themes, and a few things that make you shout...
>
> [ohmyzs.sh](https://ohmyz.sh/)

#### 下载与安装

Oh-my-zsh 官方支持 `curl` 和 `wget` 两种方式安装，看 Mac 里面安装了哪个工具就用哪个

~~后续再来介绍如何配置 `homebrew` 并利用 `brew` 来安装软件~~

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# 用 curl 安装

sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
# 用 wget 安装
```



命令执行成功后，不需要配置，直接把 `oh-my-zsh` 安装在 `~/.oh-my-zsh` 目录下，并载入默认配置：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cgoz16xrj310y0pg1hw.jpg" alt="image.png" style="zoom:100%;" />



#### 切换主题

修改主题则只需要修改 `~/.zshrc` 中的 `ZSH_THEME=""` 字段，并将相应主题下载到 `~/.oh-my-zsh/themes` 中：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5cgtc5invj311a0pa7wh.jpg)



### Powerlevel10k

俗称 `p10k`，是目前 `Oh-my-zsh` 里面最流行的主题之一，也是我最喜欢的一个主题

~~因为懒癌晚期，不喜欢自己定制配置文件，直接跑别人写好的定制脚本就行~~

#### 下载与安装

```bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
# git 一键安装
```



#### 修改 zsh 配置文件

修改之前最好提前安装前述的 `MesloLGS NF Regular.ttf`  字体。

如上述，修改 `~/.zshrc` 文件，将主题字段改为 `ZSH_THEME="powerlevel10k/powerlevel10k"`

然后保存退出，重新载入配置文件：

```bash
source ~/.zshrc
```

重载配置文件后，会自动启动 `p10k` 的安装程序：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5chay2w7vj310y0pg7ov.jpg)



#### 重置 p10k

如果上一次写入配置文件成功，直接执行以下命令，重新配置主题：

```bash
p10k configure
```

如果写入配置文件不成功，重载 `zsh` 配置文件，重新启动 `p10k` 的安装程序：

```bash
source ~/.zshrc
```



先写到这里，下次想起来再来更新



## Reference

> https://makeoptim.com/tool/terminal
>
> 