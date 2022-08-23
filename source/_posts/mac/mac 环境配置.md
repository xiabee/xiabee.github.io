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

公司发了新电脑，`M1 Pro` 的 `MacBook Pro`，用了几天体验还不错，参考 [makeoptim](https://makeoptim.com/) 配置了一下自己的终端。

这里介绍一下怎么在 `Mac` 里打造一个能让颜狗落泪的漂亮终端（划掉）

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h58pm1gs0uj30y20iu0za.jpg" alt="image.png" style="zoom:100%;" />



## “完美”标准

* 我是颜狗，终端好看是第一标准
* 自动补全应该是一个终端的必备需求
* 最好有语法高亮
* 最好能通过键盘唤醒

大概长这样：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cfxyz9ayj310y0pgqn1.jpg" alt="image.png" style="zoom:100%;" />



## 准备工作

需要安装一些软件 / 字体等

### 字体

后面会用到 `p10k` 相关字体，前往 https://github.com/romkatv/powerlevel10k#manual-font-installation 下载并安装 `MesloLGS NF Regular.ttf` 即可。

### iTerm2

#### 下载与安装

一个终端工具，新款的 Mac 应该有预装，如果没有的话可以去[官网](https://iterm2.com/)下载安装一个。

在 `Application` 目录下，找不到的话也可以 `command + [space]` 进行查找：

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cg5x52ttj311i05cjs7.jpg" alt="image.png" style="zoom:100%;" />



#### 完全磁盘访问权限

>  注：建议为 iTerm2 打开完全磁盘访问权限，避免出现默认 Terminal 能够执行正确，iTerm2 因为权限问题导致执行有误的情况。

* 左上角苹果图标 ➡️ 系统偏好设置

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cgen3sifj30ds0hcmzx.jpg" alt="image.png" style="zoom:100%;" />

* 系统偏好设置 ➡️ 安全性与隐私 ➡️ 隐私 ➡️ 完全磁盘访问权限 ➡️ iTerm

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1h5cgbwvu3gj31140wm7df.jpg" alt="image.png" style="zoom:100%;" />



#### 设置背景

* 点击屏幕左上角 `iTerm2` ➡️ `Preferences` ➡️ `Profiles` ➡️ `Profile Name`选中对应配置文件 ➡️ `Window` ➡️ `Background Image` 

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5dbp9dyggj32321cwhdu.jpg)



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



## 插件配置

这部分不是必须，但是可以让终端更好看 ✅

部分插件需要用到 `homewbrew`，后续文章会详细介绍 `homebrew`（下次一定写，这次先鸽了x）

### autojump

[autojump](https://github.com/wting/autojump) 可以记录下之前 cd 命令访过的所有目录，下次要去那个目录时不需要输入完整的路径，直接 `j somedir` 即可到达，甚至那个目标目录的名称只输入开头即可，实现了目录的自动补全。

#### 下载与安装

有两种常见的安装方式，包管理器安装和源码安装：

包管理器 `brew` 安装：

```bash
brew install autojump
```



源码安装：详情可见[官网](https://github.com/wting/autojump)

```bash
git clone git://github.com/wting/autojump.git
cd autojump
./install.py
```



#### 配置

在 zsh 的配置文件 `~/.zshrc` 中的 `plugins` 中加入 `autojump`。

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5db7ybvlzj30sg0ao7by.jpg)



然后重载 `~/.zshrc` ，启用插件

```bash
source ~/.zshrc
```



### zsh-syntax-highlighting

[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) 终端命令语法高亮插件。

执行以下命令，安装 `zsh-syntax-highlighting`

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

安装后，如上述，修改 `~./zshrc` ， 在`plugins` 中加入 `zsh-syntax-highlighting`，并重载 `~./zshrc`。



### zsh-autosuggestions

[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 终端命令自动推荐插件，会记录之前使用过的命令，当你输入开头时，会暗色提示之前的历史命令供你选择，可直接按右方向键选中该命令。

执行以下命令，安装`zsh-autosuggestions`。

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions 
```

安装后，如上述，修改 `~./zshrc` ， 在`plugins` 中加入 `zsh-autosuggestions`，并重载 `~./zshrc`。



## VSCode 相关

默认情况下，在 `VSCode` 中选择 `zsh` 作为默认 `Shell` 会出现乱码现象。原因是 `Oh My Zsh` 配置完成后，使用了 `MesloLGS NF` 字体。

因此，修复乱码只需要在设置中找到 `terminal font`，设置成 `MesloLGS NF` 即可。

* `command + ,` 打开设置 ➡️ 搜索 `terminal font` ➡️ 修改字体为 `MesloLGS NF` 

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5dbigivcdj31kg0m2n2u.jpg)



## 最终效果

`iTerm`:

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5dbk152ivj310y0pghaj.jpg)



`VScode`:

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5dbj91yqsj30ps08q406.jpg)



## Reference

> https://makeoptim.com/tool/terminal
>