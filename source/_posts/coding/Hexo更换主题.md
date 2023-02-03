---
title: Hexo更换主题
date: 2021-10-31 22:46:18
tags:  
  - hexo
  - archer
categories:
  - coding
abbrlink: hexo-archer
---



## 本站主题

* [Archer](https://github.com/fi3ework/hexo-theme-archer)
* [Demo](https://fi3ework.github.io/archer-demo/)



## 更换方式

1. 官网下载主题
2. 将主题源代码解压到`hexo`的`theme`目录下，并更改该主题的目录名称
3. 修改`hexo`根目录中的`_config.yml`文件的`theme`字段，将其改为对应主题的目录名

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gvyu8qk9j4j30t90grdot.jpg)



然后进入主题目录，找到主题的`_config.yml`进行修改即可。

最后重新渲染并部署：

```bash
hexo clean
hexo g
hexo d
```



## Hexo主题

* [官网主题](https://hexo.io/themes/index.html)

* [知乎推荐](https://www.zhihu.com/question/24422335)
