---
title: Docker搭建Markdown共享编辑器
date: 2021-11-7 20:00:23
tags:
  - markdown
  - docker
categories:
  - coding
abbrlink: markdown-docker
---





## 前情提要

* 腾讯云[双十一](https://cloud.tencent.com/act/double11?from=pre-2021double11)特惠，一次买了个`8M`带宽、`2核4G`的云服务器......不拿它整点活可惜了

* 目前的想法是搞个`markdown`共享编辑器、个人网盘啥的



## 搭建动机

主流的共享编辑器有很多，这里安利一个我个人比较喜欢的：[Hackmd](https://codimd-app.herokuapp.com/)的[Codimd](https://codimd-app.herokuapp.com/)

上述的链接需要翻墙，非常不方便，所以我想在自己的服务器里面也造一个......



## 前期准备

* 一个[VPS](https://www.zhihu.com/topic/19563791/hot)
* 一个[SSL证书](https://cloud.tencent.com/product/ssl)（可选）

* VPS中安装了`docker`、`docker-compose`等服务

```bash
sudo apt install docker
sudo apt install docker.io
sudo apt install docker-compose
sudo service docker start
# 下载、启动docker等
```





## 部署方式

* 官方推荐通过[容器部署](https://hackmd.io/c/codimd-documentation/%2Fs%2Fcodimd-docker-deployment)，这里也是介绍容器部署方式，**同时添加`nginx`代理**
* 其他部署方式可以参考[官方文档](https://hackmd.io/c/codimd-documentation/%2Fs%2Fcodimd-manual-deployment)



### 编写yml文件

创建一个空目录，在该目录中创建`docker-compose.yml`。

内容直接利用官方的：

```yml
version: "3"
services:
  database:
    image: postgres:11.6-alpine
    environment:
      - POSTGRES_USER=codimd
      - POSTGRES_PASSWORD=change_password
      - POSTGRES_DB=codimd
    volumes:
      - "database-data:/var/lib/postgresql/data"
    restart: always
  codimd:
    image: hackmdio/hackmd:2.4.1
    environment:
      - CMD_DB_URL=postgres://codimd:change_password@database/codimd
      - CMD_USECDN=false
    depends_on:
      - database
    ports:
      - "3000:3000"
    volumes:
      - upload-data:/home/hackmd/app/public/uploads
    restart: always
volumes:
  database-data: {}
  upload-data: {}
```



此时通过`docker-compose up -d`启动容器，这个服务默认端口是`3000`。在浏览器输入你的域名和端口`http://xxxx.xxx.xxx:3000`就能看到这个编辑器了。

如果看不到的话记得检查一下云服务器厂商的**防火墙**关没关......

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw6lwp9v3ij31hc0n4wm3.jpg)



当然，这个是最基础的版本，我们可以继续优化一下。



### 添加Nginx代理

目录结构：

```bash
.
├── docker-compose.yml
└── proxy
    └── conf.d
        └── hackmd.conf
```





在`docker-compose.yml`中添加关于`nginx`相关内容，大致内容如下：

```yml
version: "3"
services:
  database:
    image: postgres:11.6-alpine
    environment:
      - POSTGRES_USER=codimd
      - POSTGRES_PASSWORD=change_password
      - POSTGRES_DB=codimd
    volumes:
      - "database-data:/var/lib/postgresql/data"
    restart: always
  codimd:
    image: hackmdio/hackmd:latest
    environment:
      - CMD_DB_URL=postgres://codimd:change_password@database/codimd
      - CMD_USECDN=false
    depends_on:
      - database
    volumes:
      - upload-data:/home/hackmd/app/public/uploads
    restart: always

  proxy:
    image: nginx
    restart: unless-stopped
    expose:
      - "80"
    ports:
     - 3000:80
    # 修改端口
    volumes:
      - ./proxy/conf.d:/etc/nginx/conf.d:ro
    links:
      - codimd:codimd
    depends_on:
      - codimd

volumes:
  database-data: {}
  upload-data: {}
```



同时配置`hackmd.conf`，作为`nginx`的配置文件，大致内容如下：

```nginx
upstream @codimd {
    server codimd:3000;
    keepalive 300;
}

# for socket.io (http upgrade)
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
  listen 80;
  server_name  xxx.xxx.xx;

  location / {
    # set header for proxy protocol
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    proxy_read_timeout 300;
    proxy_connect_timeout 300;
    proxy_pass http://@codimd;
  }
}
```



然后再重新部署容器：

```bash
docker-compose down
# 删除上一个服务

docker-compose up -d
# 启动容器
```



此时我们的`nginx`代理已经开始运行了：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw6mek56brj30ih0bzama.jpg)



### 启用SSL（可选）

参考[官方文档](https://hackmd.io/c/codimd-documentation/%2F%40codimd%2FByDxZQgBL)。

#### 注册SSL证书，并将目录放置到`proxy`目录中：

```bash
.
├── docker-compose.yml
└── proxy
    ├── conf.d
    │   └── hackmd.conf
    └── ssl_certs
        ├── cert.cer
        ├── cert.key
        └── fullchain.cer
```



#### 修改`docker-compose.yml`：

```yml
version: "3"
services:
  database:
    image: postgres:11.6-alpine
    environment:
      - POSTGRES_USER=codimd
      - POSTGRES_PASSWORD=change_password
      - POSTGRES_DB=codimd
    volumes:
      - "database-data:/var/lib/postgresql/data"
    restart: always
  codimd:
    image: hackmdio/hackmd:latest
    environment:
      - CMD_DB_URL=postgres://codimd:change_password@database/codimd
      - CMD_USECDN=false
    depends_on:
      - database
    volumes:
      - upload-data:/home/hackmd/app/public/uploads
    restart: always

  proxy:
    image: nginx
    restart: unless-stopped
    expose:
      - "80"
    ports:
     - 3000:443
     # 修改端口
    volumes:
      - ./proxy/conf.d:/etc/nginx/conf.d:ro
      - ./proxy/ssl_certs:/etc/nginx/ssl_certs:ro
    links:
      - codimd:codimd
    depends_on:
      - codimd

volumes:
  database-data: {}
  upload-data: {}
```



#### 修改hackmd.conf

把域名和证书设置为自己的：

```nginx
# setup a upstream point to CodiMD server
upstream @codimd {
    server codimd:3000;
    keepalive 300;
}

# for socket.io (http upgrade)
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name your.domain.name;
    # enforce https
    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your.domain.name;

    ssl_certificate /etc/nginx/ssl_certs/cert.cer;
    ssl_certificate_key /etc/nginx/ssl_certs/cert.key;

    location / {
        proxy_http_version 1.1;
        
        # set header for proxy protocol
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        
        # setup for image upload
        client_max_body_size 8192m;
        
        # adjust proxy buffer setting
        proxy_buffers 8 32k; 
        proxy_buffer_size 32k; 
        proxy_busy_buffers_size 64k;
  
        proxy_max_temp_file_size 8192m;
        
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_pass http://@codimd;
      }
}
```



然后重新构造容器：

```bash
docker-compose down
# 删除上一个服务

docker-compose up -d
# 部署容器

docker-compose restart
# 重启容器
```



此时已经可以`https`访问了：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw6mobzd7jj31h40nk115.jpg)



## 一些BUG

### 证书相关

如果`http`能访问，但是`https`不能访问，可以检查一下证书路径是否设置正确、`nginx`监听端口是否正确等。多看`docker-compose logs`，看看具体报错

```bash
docker-compose logs
docker-compose logs --tail 10
docker-compose logs --tail 10 | grep proxy
```



### 代理相关

检查一下`nginx`的配置文件有没有写错......

之前我本人写的`xxx.conf`就有bug，一直登录不上



### 主页能访问，但是无法登陆

再次检查一下`nginx`的配置文件有没有写错......

如果完全不会写，可以直接抄这篇博客里面的，修改一个个人信息配置就行。