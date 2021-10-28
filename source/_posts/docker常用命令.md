---
title: docker常用命令
date: 2020-3-18 20:00:23
tags:
  - docker
categories:
  - coding
abbrlink: docker-commands
---



 运行docker时，发现权限不足可以康康这个：[不输入sudo运行docker](https://xiabee.cn/coding/不输入sudo运行docker/) 



### 1.查看docker容器信息

```bash
docker --help    #查看docker容器帮助
docker version   #查看docker容器版本
docker info      #查看docker容器信息
```

![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuavnv7g2j30ci0fq46o.jpg)

### 2.镜像查看

```bash
docker images     #列出本地images
docker images -a  #含中间映像层
docker images -q  #只显示镜像ID
docker images -qa #含中间映像层  

docker images --digests  ##显示镜像摘要信息(DIGEST列)
docker images --no-trunc ##显示镜像完整信息
```

![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuawgzgv5j30jq05yjyn.jpg)



### 3.镜像搜索

```bash
docker search wordpress  #搜索wordpress镜像

docker search --filter=stars=20 wordpress #只显示stars大于等于20的镜像
docker search --no-trunc wordpress        #显示镜像完整 DESCRIPTION 描述
docker search  --automated wordpress      #只显示AUTOMATED=OK 的镜像
```

![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuawnxjhxj30zs099tpf.jpg)



### 4.镜像下载

```bash
docker pull wordpress         #下载官方最新镜像，相当于 docker pull wordpress:latest
docker pull wordpress:latest  #按照tag下载镜像，这里的tag是latest
docker pull -a wordpress      #下载全部wordpress镜像
docker pull xxxx/wordpress    #下载私人仓库镜像
```

如果 pull 过慢可以参考这篇博客：[docker下载过慢：换源](https://xiabee.cn/coding/docker下载过慢：换源/)

![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuawyt4t5j30pw0dagyn.jpg)



### 5.镜像删除

```bash
docker rmi wordpress          #相当于docker rmi wordpress:latest
docker rmi wordpress:latest   #指定tag删除，单个删除
docker rmi -f wordpress       #强制删除，针对基于镜像有运行的容器进程

docker rmi -f wordpresss tomcat nginx   #多镜像删除，不同镜像间以空格间隔
docker rmi -f $(docker images -q)  #本地镜像全部删除
```



![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuaxbbjlrj30oc0b6ng1.jpg)



### 6.镜像构建

```bash
cd /docker/dockerfile   ##编写dockerfile
vim mydocker
xxxxxxxx                ##编写dockerfile
docker build -f /docker/dockerfile/mydocker -t mymydocker:1.1  ##构建docker镜像
```



### 7.容器启动

```bash
docker run -i -t --name wordpress
##新建并启动容器，参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称

docker run -d wordpress
##后台启动容器，参数：-d  已守护方式启动容器
```



### 8.容器查看

```bash
docker ps      ##查看正在运行的容器
docker ps -q   ##查看正在运行的容器的ID
docker ps -a   ##查看正在运行+历史运行过的容器
docker ps -s   ##显示运行容器总文件大小
docker ps -l   ##显示最近创建容器
docker ps -n 3 ##显示最近创建的3个容器
 
docker ps --no-trunc   ##不截断输出


docker inspect wordpress   
##获取镜像wordpress的元信息

docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' wordpress
##获取正在运行的容器wordpress的 IP
```

  注意：直接使用名称可能会找不到容器，建议直接使用容器ID进行容器操作  

![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuaxn6yl1j316o054k1i.jpg)



### 9.容器日志

```bash
docker logs xxxxx
##查看xxxxxx容器日志，默认参数

docker logs -f -t --tail=20 wordpress
##查看wordpress容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；


docker logs --since="2019-05-21" --tail=10 wordpress
##查看容器wordpress从2019年05月21日后的最新10条日志。
```

![img](https://tva1.sinaimg.cn/large/0084b03xly1gvuaxwgkq5j30mn062do9.jpg)

`docker logs 2968994e3fb2 --since="2019-05-21" --tail=10`，这里直接使用了容器ID

### 10.容器的进入与退出

```bash
docker run -it centos /bin/bash
##使用run方式在创建时进入

exit
##关闭容器并退出

docker attach --sig-proxy=false centos 
##直接进入centos 容器启动命令的终端，不会启动新进程，多个attach连接共享容器屏幕，参数：--sig-proxy=false  确保CTRL-D或CTRL-C不会关闭容器

docker exec -i -t  centos /bin/bash
##在 centos 容器中打开新的交互模式终端，可以启动新进程，参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端

docker exec -i -t centos ls -l /tmp
##以交互模式在容器中执行命令，结果返回到当前终端屏幕

docker exec -d centos  touch cache.txt 
##以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
```

这里同样建议通过ID进入容器



### 11.容器的停止与删除

```bash
docker stop wordpres    ##停止一个运行中的容器
docker kill wordpres    ##干掉一个运行中的容器
docker rm wordpres      ##删除一个已停止的容器
docker rm -f wordpres   ##删除一个运行中的容器
docker rm -f $(docker ps -a -q)   
docker ps -a -q | xargs docker rm  ##删除多个容器

docker rm -l db         ## -l 移除容器间的网络连接，连接名为 db

docker rm -v wordpres   ## -v 删除容器，并删除容器挂载的数据卷
```



### 12.生成镜像

```bash
docker commit -a="DeepInThought" -m="my wordpres" [wordpres容器ID]  mywordpres:v1.1
##基于当前wordpres容器创建一个新的镜像；参数：-a 提交的镜像作者；-c 使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
```



### 13.容器与宿主机间的数据拷贝

```bash
docker cp rabbitmq:/[container_path] [local_path]
##将rabbitmq容器中的文件copy至本地路径

docker cp [local_path] rabbitmq:/[container_path]/
##将主机文件copy至rabbitmq容器

docker cp [local_path] rabbitmq:/[container_path]
##将主机文件copy至rabbitmq容器，目录重命名为[container_path]（注意与非重命名copy的区别）
```



docker常用命令大致就这些，看起来很多，实际上掌握基本的**拖取、运行、查看日志**等操作后基本能满足日常使用需求



今天就先分享到这里，下次有空再更（划掉）

