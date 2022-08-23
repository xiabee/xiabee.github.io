---
title: Mac 环境配置(2)——brew
date: 2022-8-23 12:00:23
tags:
  - mac
  - brew
categories:
  - coding
abbrlink: mac-brew
---



## Homebrew 简介

* 官网：https://brew.sh/

* 可以理解为 `MacOS` 的包管理器，利用 `brew` 等命令实现一键安装各类软件与依赖。

* 新款 `MacOS` 应该已经预装了，如果没有的话可以参考[安装](##安装)部分。



## 安装

需要在终端中安装，如果没有配置终端可以参考[上篇](https://blog.xiabee.cn/posts/mac-init/)。

终端配置完毕后，直接在终端内执行以下命令：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装不难，如果你能顺利解决一些`BUG`的话（x）



## 常见问题

### curl SSL

```bash
curl: (60) SSL certificate problem: self signed certificate in certificate chain
More details here: https://curl.se/docs/sslcerts.html
```

* 问题描述：自签名证书不被信任，无法使用 `curl`
* 解决方案：这个没有太好的解决方案，要么重装一个受信任的证书，要么使用 `-k` 参数忽略 `https`证书校验



这里我们选择忽略校验，即直接把前述命令改为：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh -k)"
```



也可以选择一种一劳永逸的忽略校验的方法：创建一个配置文件 `~/.curlrc`， 内容为 `--insecure`

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5gs6w0r1xj307i04gaav.jpg)



### 网络问题

由于一些众所周知的原因，`GitHub` 在没有代理的时候会很难上。

#### 换源安装

这里我们直接把他改成清华源，利用国内镜像下载安装。

官方文档：https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/

```bash
xcode-select --install
# 安装 xcode-select

export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
# 设置环境变量

git clone --depth=1 https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/install.git brew-install
/bin/bash brew-install/install.sh
rm -rf brew-install
# 从本镜像下载安装脚本并安装 Homebrew / Linuxbrew
```



#### 替换仓库上游

替换 `brew` 程序本身的源

```bash
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
brew update

# 手动设置
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
brew tap --custom-remote --force-auto-update homebrew/core https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
brew tap --custom-remote --force-auto-update homebrew/cask https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
brew tap --custom-remote --force-auto-update homebrew/cask-fonts https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-fonts.git
brew tap --custom-remote --force-auto-update homebrew/cask-drivers https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-drivers.git
brew tap --custom-remote --force-auto-update homebrew/cask-versions https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-versions.git
brew tap --custom-remote --force-auto-update homebrew/command-not-found https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-command-not-found.git
brew update

# 或使用下面的几行命令自动设置
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
for tap in core cask{,-fonts,-drivers,-versions} command-not-found; do
    brew tap --custom-remote --force-auto-update "homebrew/${tap}" "https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-${tap}.git"
done
brew update
```



**注：如果用户设置了环境变量 `HOMEBREW_BREW_GIT_REMOTE` 和 `HOMEBREW_CORE_GIT_REMOTE`，则每次执行 `brew update` 时，`brew` 程序本身和 Core Tap (`homebrew-core`) 的远程将被自动设置。推荐用户将这两个环境变量设置加入 shell 的 profile 设置中。**

```bash
test -r ~/.bash_profile && echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"' >> ~/.bash_profile  # bash
test -r ~/.bash_profile && echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"' >> ~/.bash_profile
test -r ~/.profile && echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"' >> ~/.profile
test -r ~/.profile && echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"' >> ~/.profile

test -r ~/.zprofile && echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"' >> ~/.zprofile  # zsh
test -r ~/.zprofile && echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"' >> ~/.zprofile
```



#### 复原仓库上游

```bash
# brew 程序本身，Homebrew / Linuxbrew 相同
unset HOMEBREW_BREW_GIT_REMOTE
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew

# 以下针对 macOS 系统上的 Homebrew
unset HOMEBREW_CORE_GIT_REMOTE
BREW_TAPS="$(BREW_TAPS="$(brew tap 2>/dev/null)"; echo -n "${BREW_TAPS//$'\n'/:}")"
for tap in core cask{,-fonts,-drivers,-versions} command-not-found; do
    if [[ ":${BREW_TAPS}:" == *":homebrew/${tap}:"* ]]; then  # 只复原已安装的 Tap
        brew tap --custom-remote "homebrew/${tap}" "https://github.com/Homebrew/homebrew-${tap}"
    fi
done

brew update
```

**注：重置回默认远程后，用户应该删除 shell 的 profile 设置中的环境变量 `HOMEBREW_BREW_GIT_REMOTE` 和 `HOMEBREW_CORE_GIT_REMOTE` 以免运行 `brew update` 时远程再次被更换。**



## 最终效果

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h5gsvqocrtj30nm0pgqei.jpg)



## brew 常见用法

```bash
brew update
# 更新源
brew upgrade
# 下载更新
brew search xxxx
# 搜索 xxxx
brew install xxxx
# 安装 xxxx
brew remove xxxx
# 卸载 xxxx
```

* 更多用法参考[官方文档](https://docs.brew.sh/)



## Reference

> https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/
>
> https://docs.brew.sh/