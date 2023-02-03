---
title: Docker入门
date: 2020-3-17 20:00:23
tags:
  - docker
categories:
  - coding
abbrlink: docker-pupy
---



## Docker简介

* [官方文档](https://docs.docker.com/)
* 简言之：开源的、跨平台的、虚拟化的应用引擎；使用沙箱机制。
* 一个完整的Docker通常会包括以下几个部分：

  * `DockerClient` 客户端
  * `Docker Daemon`守护进程
  * `Docker Image`镜像
  * `DockerContainer`容器 


![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gw7zqqkvyzj31gx0acdw0.jpg)



### Docker与VM

可能很多朋友都用过虚拟机，而对容器这个概念比较的陌生——简单来说，可以理解为：**相较于VM，docker是轻量级的虚拟化技术，却有着更强大的性能**。



#### 虚拟机（VM）

- 我们用的传统虚拟机如 [VMware](https://www.vmware.com/cn.html) ， [virtualbox](https://www.virtualbox.org/)之类的需要模拟整台机器包括硬件。
- 每台虚拟机都需要有自己的操作系统，虚拟机一旦被开启，预分配给它的资源将全部被占用。
- 每一台虚拟机包括应用，必要的二进制和库，以及一个完整的用户操作系统。



#### 容器（docker/Container）

- 容器技术是和我们的**宿主机共享硬件资源及操作系统**，可以实现资源的动态分配。
- 容器包含应用和其所有的依赖包，但是**与其他容器共享内核**。容器在宿主机操作系统中，在用户空间以分离的进程运行。
- 容器技术是实现**操作系统虚拟化**的一种途径，可以让您在资源受到隔离的进程中运行应用程序及其依赖关系。
- 通过使用容器，我们可以轻松打包应用程序的代码、配置和依赖关系，将其变成容易使用的构建块，从而实现环境一致性、运营效率、开发人员生产力和版本控制等诸多目标。
- 容器可以帮助保证应用程序快速、可靠、一致地部署，其间不受部署环境的影响。
- 容器还赋予我们对资源更多的精细化控制能力，让我们的基础设施效率更高。 

一张图展示Docker和VM的区别：

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gw7yf4hkwsj30ou0ajwhn.jpg)

左图为`dokcer`，右图为`VM`



## Docker安装

` docker`分社区版和企业版，这里我们选择社区版（因为企业版要钱） 

官方文档中各大环境的安装说明：[ https://docs.docker.com/install/ ](https://docs.docker.com/install/)



### 包管理器安装

这里我们以Ubuntu为例：

正常的话用包管理器就装完了：

```bash
sudo apt update
sudo apt install docker 
sudo apt install docker.io
```



### 其他安装

* 参考[知乎](https://zhuanlan.zhihu.com/p/54147784)等教程



### 检测版本

```bash
docker -v
```

得到这种显示则说明安装成功：

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gw7ylhg1axj30g002qdgz.jpg)





## Docker换源

* 参考[这篇博客](https://blog.xiabee.cn/posts/docker-source)



## docker-compose

* [官网](https://docs.docker.com/compose/)

* 用于编排容器，具体教程以后再写（x）

```bash
sudo apt install docker-compose
```



## 关于其他

* 常用命令可以参考这篇博客：[docker常用命令](https://blog.xiabee.cn/posts/docker-commands/)

* 运行`docker`时发现权限不足可以康康这个：[不输入sudo运行docker](https://blog.xiabee.cn/posts/docker-sudo)

