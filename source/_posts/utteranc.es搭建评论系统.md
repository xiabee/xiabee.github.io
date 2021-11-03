---
title: utteranc 搭建评论系统
date: 2021-11-3 22:46:18
tags:  
  - hexo
  - utteranc
categories:
  - coding
abbrlink: utteranc-hexo
---



## 简介

* [官网](https://utteranc.es/)
* 利用`github issue`做静态博客的评论系统

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw26hn9pp6j30rk0a5dh7.jpg)



## 关于Gitalk

`gitalk`曾经（包括现在）也是很火的博客评论插件，但是它出现过安全事故:[v2ex论坛](https://www.v2ex.com/t/535608)， [黑客派骗star](https://www.v2ex.com/t/534800?p=2)，就不推荐使用。

这里给不能科学上网的同学简述一下：这个项目要的权限太多了，**最坏情况下**，恶意的使用者可以直接修改你的项目......黑客派就曾利用你登录评论区的`token`，偷偷的给自己仓库标`star`......

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1gw26w6dt86j30p90o412i.jpg)





<img src="https://tva1.sinaimg.cn/large/0084b03xgy1gw26pr4jzuj30ji0g5jv8.jpg" alt="image.png" style="zoom: 80%;" />





## 关于utteranc

<img src="https://tva1.sinaimg.cn/large/0084b03xly1gw2700msb4j30eo0nhq62.jpg" alt="image.png" style="zoom: 67%;" />

* 仅用于登录`Github`，如果网页内已经登录了`Github`则无需重复授权
* 只对相关`issue`有读写权限，没有整个仓库读写权限，不存在直接利用`token`修改仓库的情况



## 搭建

### 手动搭建

[官网](https://utteranc.es/)写的很详细了，我简述一下（x）

* 创建一个仓库
* 给仓库安装 [utterances](https://github.com/apps/utterances) APP
* 配置一下读写权限

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1gw2779my61j30ia0le780.jpg" alt="image.png" style="zoom: 80%;" />

* 插入代码中：

  ```html
  <script src="https://utteranc.es/client.js"
          repo="[ENTER REPO HERE]"
          issue-term="pathname"
          theme="github-light"
          crossorigin="anonymous"
          async>
  </script>
  ```



### Archer主题集成

* 创建仓库和安装APP和手动搭建相同
* 找到主题目录的`_config.yml`中的`comment`字段：

```yml
comment:
  utteranc_repo: "[ENTER REPO HERE]" # 这里填你的repo
  utteranc_label: "Comment"
  utteranc_theme: "github-light"
  utteranc_issue_term: "title"
```



## 允许/禁止comment

在`md`文件首部`YAML Front Matter`部分添加`comments`字段，例如：

```yml
title: utteranc 搭建评论系统
date: 2021-11-3 22:46:18
tags:  
  - hexo
  - utteranc
categories:
  - coding
```



## 测试

### 评论测试

发布评论：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw27jubr7yj30s90df0ul.jpg)



邮件提醒：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw27kqwm16j30m80ex7bk.jpg)



`ISSUE`：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1gw27lhsg4tj30zs0ljthv.jpg)



### 修改提醒

如果想修改邮件提醒可以在右上角`Notifications`里面改：

![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1gw27n0sqcaj30e00dx77c.jpg)



![image.png](https://tva1.sinaimg.cn/large/0084b03xgy1gw27ne3uqdj30e00ckgnj.jpg)