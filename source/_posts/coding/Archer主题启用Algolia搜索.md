---
title: Archer主题启用Algolia全文搜索
date: 2021-10-31 23:48:18
tags:  
  - hexo
  - archer
  - algolia
categories:
  - coding
abbrlink: archer-algolia
---



## 问题描述

* `Hexo`缺少站内搜索功能
* `Hexo`自带的`tag`功能并不能满足搜索需求
* `Archer`主题自带的搜索模块不能实现全文检索（详情可以参考相关[issue](https://github.com/fi3ework/hexo-theme-archer/issues/243)）



`Hexo`本质上是一个静态页面的渲染工具，而我们的博客部署在`Github Page`上，没有数据库的操作权限，也就不能像`Wordpress`那样自身实现站内的高级搜索......然而主题自带的第三方搜索插件又无法检索**文章内容**......



## 解决方案

* 利用`Archer`封装的第三方插件[Algolia](https://www.algolia.com/)实现搜索
* 利用`hexo-algoliasearch`实现全文检索

> * 为什么不直接用hexo-algoliasearch做搜索
> * 因为主题没有封装，裸着搜索很违和，自己封装又好累(x)



## Archer主题的algolia设置

### Algolia简介

* [官网](https://www.algolia.com/)
* 提供云搜索
* 白嫖用户可上传`10,000`条`JSON`数据
* 白嫖用户每个月可操作共`100,000`次（上传、搜索）



### 注册Algolia

* 直接[官网](https://www.algolia.com/)注册即可
* 注册之后会创建一个应用，并得到一些`API Keys`，记住这些`API Keys`，一会需要用到

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvyvkz1s5bj31hc0p2tjh.jpg)

在`Applications`中能看到刚刚申请的应用`id`，`API Keys`里面有需要的`API key`



![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvyvovwaskj31hc0p2akj.jpg)



* 注意：不要在任何配置文件中填写`Admin API Key`！如果有必要，则使用全局变量的方式替代



### 修改Hexo根目录的_config.yml

搜索`algolia`模块，如果没有则自己添加一个：

```yml
# searching 
algolia:
  applicationID: 'xxxx' # used for hexo-algolia
  appId: 'xxxx'         # used for hexo-algoliasearch
  apiKey: 'xxxxxxx'
  chunkSize: 5000
  indexName: 'xxxxxx'
  fields:
    - path
    - title
    - content:strip
    - excerpt:strip
 # fields 这栏根据自身需求设置
```



这里的`applicationID`和`appId`其实是一个东西，就是刚刚创建的应用的`ID`，只是我们需要利用**两个不同的插件**，所以写了两遍。



### 安装相关插件

在`Hexo`根目录下执行：

```bash
npm install hexo-algolia --save
# 这个是主题集成的搜索模块所需插件，不支持搜索文章内容

npm install hexo-algoliasearch --save
# 这个是我们搜索全文用到的插件
```



### 修改主题_config.yml

`Archer`主题自身集成了`algolia`模块，把主题的`_config.yml`中的`algolia_search`模块的`false`改成`true`即可

```yml
# ========== Search ========== #
algolia_search:
  enable: true
  hits:
    per_page: 10
  labels:
    input_placeholder: Search for Posts
    hits_empty: "We did not find any results for the search: ${query}"
    hits_stats: "${hits} results found in ${time} ms"
```

* 注意：需要先安装相关插件再启用`algolia`模块，否则在渲染时会缺少`js`文件导致程序报错。



## hexo-algoliasearch相关设置

全部设置可以参考[官网](https://github.com/LouisBarranqueiro/hexo-algoliasearch)



### 安装模块

`hexo-algoliasearch`刚刚已经安装过了，就不重复安装了



### 修改_config.yml

在`Hexo`根目录的`_config.yml`文件中添加`plugins`字段：

```yml
plugins:
  - hexo-algoliasearch
```



### 参数设置

* 不要像[官网](https://github.com/LouisBarranqueiro/hexo-algoliasearch)一样把`adminApiKey`写进配置文件里！（因为总有人会顺手把配置文件传到`Github`，比如我）

* 在不填写`adminApiKey`的时候，可以通过环境变量的方式认证`adminApiKey`（`Windows`用户手动添加`PATH`即可）

  ```bash
  export ALGOLIA_ADMIN_API_KEY=xxxxxxxx
  # 你的 admin_api_key
  
  export HEXO_ALGOLIA_INDEXING_KEY=xxxxxxx
  # 你的Search-Only API Key
  ```

PS：`Search-Only API Key`其实就是配置文件中的`apiKey`，但是有时候`hexo-algoliasearch`模块莫名其妙找不到这个`Search-Only API Key`，可以通过上面的操作解决。



## 更新Algolia

如果你不是`WSL`用户，这时直接`hexo aloglia`即可。

如果是`WSL`用户，则需要检查一下环境变量有没有被覆盖......最懒的办法就是每次都添加一遍（不是）

```bash
export ALGOLIA_ADMIN_API_KEY=xxxxxxxx
# 你的 admin_api_key

export HEXO_ALGOLIA_INDEXING_KEY=xxxxxxx
# 你的Search-Only API Key

hexo algolia
# 更新 algoia
```

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvyx08rau1j30ol0dzdpl.jpg)



此时在`Algolia`管理界面里面已经能看到文章内容了：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvyxan0iqaj31hc0p24bc.jpg)



## 搜索效果

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gvyx4fhdlsj30pa0kt0ve.jpg)



标题包含关键词的会标红，**文章内容**包含关键字也能搜索到
