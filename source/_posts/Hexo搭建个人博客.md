---
title: Hexo+GitPage搭建个人博客
date: 2021-10-27 23:46:18
tags:  
  - hexo
  - git
categories:
  - coding
abbrlink: hexo-git-setup
---



## 前言

之前一段时间都是用`wordpress`做[个人博客](https://xiabee.cn)，后来发现`Git Page + hexo`可以免费做静态页面......尝试了一下，后面就真香了（x）

关于`wordpress`和`hexo`孰优孰劣的问题这里不做详细讨论，只是介绍一下如何用`Git Page`+`hexo`搭建一个个人博客。二者具体的比较可以参考[知乎](https://www.zhihu.com/question/53068081)的激(you)烈(hao)讨论。



## 简介

### What's GitHub Page

* [官方文档](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages)

* 简而言之就是 *白嫖* `GitHub`的服务器，通过仓库挂一个自己的静态网站。



### What's Hexo

* [官方文档](https://hexo.io/zh-cn/)

* 一个渲染静态博客的框架，基于`Node.js`，将`markdown`文件渲染成`html`文件



### Hexo + Github Page

* 利用`hexo`快速生成需要的页面，利用`github`在公网展示出来



## 创建Github Pages

创建一个公网能访问的`page`，让大家能看到你的博客，这里利用`Github page`

### 创建repo

创建一个名为`xxx.github.io`的仓库，且允许公网访问：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvwe3g2y4pj30px0c8ade.jpg)



### 创建Pages

在里面随便新建一个文件，确保`main`分支有东西。

然后在`Settings -> Pages`里面设置分支和网站根目录：这里直接设置为`main`和`/(root)`。如果网页上传在其他分支上可以在这里修改。

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvwe8mpskbj31720n0wot.jpg)



此时访问`https://test.github.io`应该就能看到刚刚的网页了。~（因为我已经开了一个`xiabee.github.io`，而免费版只能开一个网页，所以`test.github.io`就暂时无法展示）~





## 环境搭建

### Node.js

* `Linux`用户：直接包管理器安装即可

  ```bash
  # 例如Ubuntu
  apt update
  apt install nodejs
  ```

  包管理器找不到`nodejs`的可以和`Windows`用户一样取官网安装。

* `Windows`用户：[官网](https://nodejs.org/zh-cn/)下载并安装



### NPM

* `node.js`一般自带`npm`，但是可能不是最新版的`npm`，直接更新即可

  ```bash
  apt-get install nodejs-legacy
  apt-get install npm
  npm install -g n
  # 安装n模块
  
  n stable
  # 升级nodejs到最新stable版本
  
  npm install npm@latest -g
  # 升级最新npm
  ```

* 如果没有自带`npm`可以去[官网](https://docs.npmjs.com/)看看，手动安装`npm`



### Git

* `Linux`：直接包管理器安装

  ```bash
  # ubuntu
  apt update
  apt install git
  ```

* `Windows`：[官网](https://git-scm.com/downloads)下载安装

* 配置`ssh`登录可以参考[这篇博客](https://xiabee.github.io/posts/github-ssh/)



### HEXO

先装`hexo-cli`：

```bash
sudo npm install -g hexo-cli
# 全局安装hexo
```





## 本地调试

### hexo根目录

选择一个空目录做为`hexo`的根目录，初始化并安装相关依赖：

这里建议把刚刚创建的`Github Page`的`repo`克隆下来，再把该目录删空作为根目录......不然可能会遇到分支没有共同基点不能合并的问题。

```bash
hexo init 
# 在该目录下初始化

npm install
# 安装依赖
```



### 常用hexo命令：

```bash
hexo g # hexo generate
# 渲染页面，生成网页

hexo s # hexo server
# 启动本地服务器，提供预览和本地调试

hexo clean
# 清理本地网页
```



访问[http://localhost:4000](http://localhost:4000)，出现`hexo`页面说明渲染成功

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvwdzcr2n8j31hc0n4aw0.jpg)

这里我修改过主页，所以可能看起来和初始模板不太一样。

**Tips：**如果`server`一直起不来，可能是端口被占用了。`Ctrl C` 关闭服务器，运行 `hexo server -p 5000` 更改端口号后重试。



### hexo常见目录结构

```bash
├── _config.yml 	# 网站配置信息
├── node_modules	# nodejs组件
├── package.json
├── package-lock.json
├── public			# 生成的网站文件
├── README.md		
├── scaffolds		# 模板文件夹
├── source			# 写博客的markdown源文件
└── themes			# 主题文件夹
```





## 远程推送

把`hexo`推送到`github`中



### 安装hexo-deployer-git

```bash
npm install hexo-deployer-git --save
```



### 修改_comfig.yml

找到`hexo`根目录下的`_config.yml`，文末`deploy:`模块：

```bash
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: 
    github: git@github.com:xiabee/test.github.io.git
    branch: main
    gitee: git@gitee.com:xiabee/test.gitee.io.git
    branch: main
```

这里是做了`Gitee+GitHub`双备份，如果不传`gitee`的话可以只写`github`的仓库地址。

此时指定`hexo`上传的网页文件为`main`分支，如果刚刚`Github Page`的分支不是`main`的可以修改一下......改哪个都可以，一致即可。



![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1gvwerqrvzfj30is07hmzf.jpg)

这里`GitHub Pages`用的就是`main`分支，不需要修改



### 推送

```bash
hexo g 
# 渲染页面

hexo d
# 推送至git，部署发布

# 二者结合相当于 hexo g -d
```



此时访问公网地址，已经能看到该博客了：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvweyd9xbzj31hl0sv1kx.jpg)



## 写文章

### 编辑源文件

在`_posts`目录下创建`markdown`文件，仿照`Hello World`写就行：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvwf1noz1nj30mb08tafh.jpg)

然后`hexo g -d`，渲染、推送。



### 关于图床

如果是整个媒体库都上传到`Github`上的话，相当于是把`Github`作为图床了......然而众所周知`Github`在国内的速度非常非常慢。所以建议换一个好用的图床。



关于图床的推荐可以参考一下[知乎](https://www.zhihu.com/question/21349585)。



这里我个人使用的是[微博](https://weibo.com/)图床，配合`Chrome`插件[新浪微博图床](https://chrome.google.com/webstore/detail/%E6%96%B0%E6%B5%AA%E5%BE%AE%E5%8D%9A%E5%9B%BE%E5%BA%8A/fdfdnfpdplfbbnemmmoklbfjbhecpnhf?utm_source=chrome-ntp-icon)使用，可以直接微信截图到剪贴板，然后复制粘贴，灰常方便：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvysn7j1qaj30ra0ivabw.jpg)



当然，用这个图床的前提是你得有一个微博账号（x）



## Hexo备份

* 利用`Git`分支
* 参考[这篇博客](https://xiabee.github.io/posts/hexo-git-backup/)

