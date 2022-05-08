---
title: Docker搭建Hydro-OJ系统
date: 2022-4-29 20:00:23
tags:
  - oj
  - docker
categories:
  - coding
abbrlink: hydro-docker
---

## Hydro简介

- `Hydro`是目前来看比较新的一款开源`Online Judge`系统
- 目前[官网](https://hydro.js.org/)有常见的安装方式，但是`Docker`的支持不是很好，于是我自己在官网架构基础上，重新编写了一下容器部署。
* 项目地址：[Hydro-Docker](https://github.com/xiabee/Hydro-Docker)

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h1r032ci9pj31ep0owto6.jpg)

## 使用方式

* 直接使用`docker-compose`运行容器，在本地构建镜像
  
  ```bash
  git clone https://github.com/xiabee/Hydro-Docker
  cd Hydro-Docker
  docker-compose up -d
  ```

* 没有报错就是成功（x）：
  
  ```bash
  docker-compose ps
  ```
  
  ![image.png](https://tva1.sinaimg.cn/large/0084b03xly1h1r0da6x26j30tl055djc.jpg)

## 聊聊编写过程

* 准确的说所有模块的测试完全靠自己，全网没有找到相关的报错，可能是因为官方并不打算提供容器安装的技术支持......

* 于是有了以下内容（都是我遇到的bug，大家构建过程中可以参考一下

### 构建速度较慢

* 问题描述：如题

* 解决方案：这个直接给`Dockerfile`换源就好，找一个合适的源，比如清华源：
  
  ```dockerfile
  RUN sed -i "s/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g" /etc/apt/sources.list && \
      sed -i "s/security.debian.org/mirrors.tuna.tsinghua.edu.cn/g" /etc/apt/sources.list && \
      apt-get update && apt-get install -y --no-install-recommends && apt upgrade -y
  ```

  同时给宿主机配置容器加速器，这里以阿里云、`Ubuntu`为例：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ee1z0bm0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 测评机不响应

* 问题描述：各个容器运行正常，但是测评机一直处于不工作状态，即提交代码无法测评
  
  #### Error: Request failed with status code 404
  
  `docker-compose logs | grep judge`查看测评机容器日志，发现以下内容：
  
  ```bash
  oj-judge    | Unhandled Rejection at: Promise  Promise {
  oj-judge    |   <rejected> Error: Request failed with status code 404
  oj-judge    |       at createError (/usr/local/share/.config/yarn/global/node_modules/axios/lib/core/createError.js:16:15)
  oj-judge    |       at settle (/usr/local/share/.config/yarn/global/node_modules/axios/lib/core/settle.js:17:12)
  oj-judge    |       at IncomingMessage.handleStreamEnd (/usr/local/share/.config/yarn/global/node_modules/axios/lib/adapters/http.js:322:11)
  oj-judge    |       at IncomingMessage.emit (events.js:412:35)
  oj-judge    |       at endReadableNT (internal/streams/readable.js:1334:12)
  oj-judge    |       at processTicksAndRejections (internal/process/task_queues.js:82:21) {
  oj-judge    |     config: {
  oj-judge    |       transitional: [Object],
  oj-judge    |       adapter: [Function: httpAdapter],
  oj-judge    |       transformRequest: [Array],
  oj-judge    |       transformResponse: [Array],
  oj-judge    |       timeout: 30000,
  oj-judge    |       xsrfCookieName: 'XSRF-TOKEN',
  oj-judge    |       xsrfHeaderName: 'X-XSRF-TOKEN',
  oj-judge    |       maxContentLength: -1,
  oj-judge    |       maxBodyLength: -1,
  oj-judge    |       validateStatus: [Function: validateStatus],
  oj-judge    |       headers: [Object],
  oj-judge    |       baseURL: 'http://oj-backend:8888/',
  oj-judge    |       method: 'post',
  oj-judge    |       url: 'login',
  oj-judge    |       data: '{"uname":"root","password":"rootroot","rememberme":"on"}'
  oj-judge    |     },
  ```

   注意此时报错：`oj-judge`在不断请求`http://oj-backend:8888/`，但是返回值一直是`404`，然而观察到`oj-backend`并没有暴露`8888`端口......发现问题所在！

* 解决方案：在`docker-compose`中暴露`oj-backend`的服务端口：
  
  ```yml
  # main site
    oj-backend:
      build: ./backend
      container_name: oj-backend
      ...
      expose:
        - 8888
  ```

#### server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]

`docker-compose logs | grep backend`查看后端容器日志，发现以下内容：

```yml
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
oj-backend  | [E]  server  User: 0(Guest) get: /judge/files You don't have the required privilege. [ 512 ]
```

此时推断，测评机在抓取后端请求，但是后端并没有权限写入测评机

结合上面测评机容器请求内容：

```yml
oj-judge    |       url: 'login',
oj-judge    |       data: '{"uname":"root","password":"rootroot","rememberme":"on"}'
```

推测测评机设置的账号密码与后端不匹配

* 解决方案：修改`./data/judge/judge.yaml`将`uname`和`password`修改为对应管理员账号的用户名和密码：
  
  ```yaml
  hosts:
    localhost:
      type: hydro
      server_url: http://oj-backend:8888/
      uname: root
      password: rootroot
      detail: true
  ```

* 然后重启容器就可以正常测评了：`docker-compose restart`

<img title="" src="https://tva1.sinaimg.cn/large/0084b03xly1h1s1y34q16j30cu06jwfl.jpg" alt="image.png" data-align="center">







### 其他问题

* 暂时不记得了，想起来了来写
