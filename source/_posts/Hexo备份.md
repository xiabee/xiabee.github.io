---
title: 利用git分支进行Hexo备份
date: 2021-10-28 23:46:18
tags:  
  - hexo
  - git
categories:
  - coding
abbrlink: hexo-git-backup
---



`hexo`博客搭建可以看[这里](https://xiabee.github.io/posts/hexo-git-setup/)。

## 问题描述

博客搭建环境：`hexo`+`gitpage`，本地编写·`markdown`文件，通过`hexo g -d`渲染后上传至`github repo`。

相较于`wordpress`，`hexo`框架的博客编写基本是在本地完成，在其他机器写博客时需要备份并重新配置环境。但是`nodejs`环境文件很多，备份起来相当麻烦。



## 解决方案

大致有两种方案：

* 创建新仓库，备份源文件
* 创建新分支，备份源文件





## 创建备份

这里我们主要介绍`github`分支的方式：主分支用于渲染`github page`，分分支用于备份重要源文件。

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvvhwdb12oj30ay0913zq.jpg)

创建新仓库的方式同理，就不介绍了（x）



### 在hexo根目录下创建git

最新版的`hexo`在初始化之后是没有`.git`目录的，通过`_config.yml`里面的`deploy`参数生成`.deploy_git`目录，并在通过该目录进行`push`操作。因此，我们直接在`hexo`根目录下创建`.git`目录和`.gitignore`并不会冲突。

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvvhx5uhi3j30ly0cqdm6.jpg)



创建本地`git`的方式有很多种，只需要将`git`的指向我们`xxx.github.io`的`repo`就行。

比如可以直接将`xxx.github.io`的整个`repo`克隆下来，然后复制其中的`.git`目录到`hexo`的根目录下。



### 创建.gitignore

常见的需要备份的文件大致有：

```bash
scaffolds/
source/
themes/
.git/
.gitignore
_config.yml
package.json
```



然而，每次筛选很麻烦，这里我们直接在`hexo`根目录下创建`.gitignore`排除剩下的不需要备份的文件。



示例如下：

```bash
# .gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
.git*/
themes/
!themes/archer/_config.yml
# 这里只备份主题的配置文件
.vscode/
```





### 创建分支

在刚刚创建好`git`的`hexo`的根目录下执行：这里以 `hexo_backup` 分支为例

```bash
git branch -a
# 查看所有分支

git branch 
# 查看当前使用分支

git checkout hexo_backup
# 切换到 hexo_backup 分支上，若不存在则创建该分支并切换到该分支

# git branch -d hexo_backup
# 删除本地 hexo_backup 分支

# git push origin --delete hexo_backup
# 删除远程 hexo_backup 分支
```

确保新分支不要与主分支重名......尽量不要取`master`、`main`这种名字就好。



## 提交备份

因为之前写了`.gitignore`，我们可以放心大胆的将当前目录全部`commit`：

```bash
git add .
# 添加当前目录

git commit -m "backup"
# 添加commit，引号内的内容随意

git push origin hexo_backup
# 将本地数据推送到 hexo_backup 分支中
```

远程仓库已有两个分支，因此必须指定分支`hexo_backup`，不能直接`git push`。

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvvi0lcpq3j30qh08m0zg.jpg)

此时本地分支和远程分支匹配



![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvvi1iy2zmj30w60koqah.jpg)

此时已经备份完毕了。



## 迁移与复原

### 新环境配置

```bash
npm install -g hexo-cli
hexo init
```





### 克隆备份分支

将原来的`source`,  `package.json`等文件克隆到`hexo`根目录下

```bash
git clone -b hexo_backup git@github.com:[username]/[username].github.io.git
npm install
```



### 渲染与推送

```bash
hexo clean
hexo g -d
# 生成新的页面并推送至主分支
```

主分支利用`hexo`自身集成的`git`组件进行推送：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvvi31gsgmj30iq0bhwmp.jpg)
