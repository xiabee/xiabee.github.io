---
title: Hexo 本地迁移
date: 2022-8-14 12:00:23
tags:
  - hexo
  - linux
  - algolia
categories:
  - coding
abbrlink: hexo-transfer-issue
---

## 前置知识

* 利用 hexo 和 gitpage 搭建博客：https://blog.xiabee.cn/posts/hexo-git-setup/
* 利用 git 分支进行源码备份：https://blog.xiabee.cn/posts/hexo-git-backup/
* 利用 algolia 启用全文搜索：https://blog.xiabee.cn/posts/archer-algolia/



## 迁移过程

### 源码下载

```bash
git clone git@github.com:xiabee/xiabee.github.io.git
# 克隆 GitPage 的仓库到本地，此时是展示 Page 页面的主分支
cd xiabee.github.io

git checkout hexo_backup
# 切换到源码备份分支
git pull
# 下载源码
```

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h57jodmddaj311k0v04qp.jpg)



### 环境安装

* 这里用的是 `M1 Pro MacbookPro`，并且安装了 `homebrew`
* 其他环境参考 [前述博客](https://blog.xiabee.cn/posts/hexo-git-backup/#/%E8%BF%81%E7%A7%BB%E4%B8%8E%E5%A4%8D%E5%8E%9F) 的“迁移与复原”部分

```bash
brew update
brew install hexo
# 在本地安装 hexo

cd xiabee.github.io
# 进入仓库目录
npm install
# 安装相关依赖
```





## 故障排查

### 页面渲染失败

#### 问题描述

* `hexo g -d` 之后，博客全白，什么也没有

#### 解决方案

* 检查一下备份文件里面有没有备份主题文件......没有的话重新下载一下主题文件，然后重新 `hexo g`



### db.json 读写有问题

#### 问题描述

* ERROR Database load failed. Deleting database.

​		读取数据库文件 `db.json` 失败，删除了  `db.json`

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h56q8ix62pj30tc0a6n64.jpg)



* OperationalError: ENOENT: no such file or directory, open '/Users/xiabee/Desktop/GitHub/gitpage/db.json'

​		找不到数据库文件 `db.json`，导致操作失败

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h56qa7jjfxj317y0ls4oa.jpg)



#### 解决方案

* 折腾了很久，只在一个古老 [issue](https://github.com/hexojs/hexo/issues/1120) 中找到了类似问题，但是并没有合适的解决方案——然后尝试了一下更新 `npm`，就莫名其妙解决了......

* ```bash
  cd xiabee.github.io
  # 进入 hexo 根目录
  npm update
  ```



### 最终效果

执行成功：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1h57kiiw6h4j310y0u64qp.jpg)

