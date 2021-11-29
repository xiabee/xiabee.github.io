---
title: Docker搭建NextCloud个人网盘
date: 2021-11-9 20:00:23
tags:
  - docker
  - nextcloud
categories:
  - coding
abbrlink: nextcloud-docker
---



## 开端

* 百度云非会员太慢了！！！
* `One Drive`有时候连不上服务器！！！
* 正好手头有个`4G`内存`8M`带宽的服务器，可以整个活
* 手头也有个域名，已经搞了`SSL`证书，可以满足网盘加密传输的需求
* 太长不看版：代码在[这里](https://github.com/xiabee/nextcloud-docker/)，设置密码、添加证书、修改域名，`docker-compose up -d`就行



## NextCloud简介

* [官网](https://nextcloud.com/)

* `nextcloud`是一款开源的、支持多平台的云盘，有服务器的小朋友可以整一个玩玩，可以搭一个自己的`One Drive`



## 搭建方式

官网的搭建过程比较麻烦，需要手动配置服务器再执行相关代码等等。这里懒癌晚期患者直接去[dockerhub](https://hub.docker.com/_/nextcloud)找了个官方镜像，直接通过容器进行安装。

首先要确保你的`VPS`中安装了`docker`和`docker-compose`

```bash
sudo apt install docker.io
sudo apt install docker
sudo apt install docker-compose
sudo service docker start
# 以 Ubuntu 为例
```



### 单容器搭建

`nextcloud`独立容器版本使用的是`Apache`服务器，自带`SQLite`作数据库，基本满足个人需求。如果对性能要求不高的话直接单容器搭建也可以：

```bash
docker run -d \
  --restart=always \
  --name nextcloud \
  -p 5000:80 \
  -v ~/nextcloud:/var/www/html \
  docker.io/nextcloud
```



参数解释：

```bash
run					# 运行容器实例
-d					# 守护模式运行
-restart=always		# 遇到问题就重启
--name nextcloud	# 命名为nextcloud
-p 5000:80			# 将容器的80端口映射到宿主机的5000端口
-v ~/nextcloud:/var/www/html	# 将容器的/var/www/html目录映射到主机的~/nextcloud目录
docker.io/nextcloud	# 创建容器用到的镜像
```

构建完毕后，通过`http://ip:5000`就能访问`nextcloud`了，然后进行相关设置即可。



### 满血搭建

* 项目在[GitHub仓库](https://github.com/xiabee/nextcloud-docker/)中
* 相较于单容器的`nextcloud`，这次满血搭建添加了`fpm`、`nginx`、`mariadb`、`redis`以提高安全性和性能
* `docker-compose`部分借鉴了一位素不相识的名为[chensmallx](https://hexo.chensmallx.top/2021/04/08/nextcloud-on-docker/)大佬的博客，稍微修改了一部分，解决了数据库的相关`BUG`
* 要实现`https`访问的话需要提前准备域名和证书

项目的目录结构如下：

```bash
├── docker-compose.yml
└── proxy
    ├── conf.d
    │   └── nextcloud.conf
    └── ssl_certs
        ├── cert.cer
        ├── cert.key
        └── fullchain.cer
```

没有域名和证书的朋友可以翻到[这里](#关闭https)

#### 搭建方法

* 将项目克隆到本地，把自己的`SSL`证书放入`proxy/ssl_certs`目录下
* 修改`proxy/conf.d/nextcloud.conf`文件，将域名换成自己的
* `docker-compose up -d`等待容器启动



#### 各模块功能

* `FPM`：`fastCgi`做为呈现层，提高处理速度
* `nginx`：前置反向代理，提高并发处理能力，同时提供`https`服务
* `mariadb`：中大型关系数据库，替换轻型的`SQLite`，提高数据读写性能和可靠性
* `redis`：缓存数据库，提供数据缓存能力和文件锁管理器



#### 依赖关系

* `nextcloud`是主业务，读写数据时会用到数据库`mariadb`和缓存`redis`，因此`nextcloud`依赖于`mariadb`和`redis`
* `mariadb`和`redis`各司其职，挂了一个不会影响到另一个，故没有依赖关系
* `nginx`作为反向代理，需要主体业务`nextcloud`正常运行才能提供代理服务，故`nginx`依赖于`nextcloud`

整体依赖关系如下：

```bash
# 依赖关系如下
                     /-> mariadb
nginx -> nextcloud -|
                     \-> redis


redis & mariadb -> nextcloud -> nginx
# 启动顺序则需要反过来
```



### 代码说明

#### 环境变量

在老版本的`docker-compose`文件中，常见的环境变量可以直接用`environment`的变量表示，类似于[chensmallx](https://hexo.chensmallx.top/2021/04/08/nextcloud-on-docker/#docker-compose%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)这种写法：

```yml
environment:
    # environment可以对容器创建指定多个环境变量
     - MYSQL_ROOT_PASSWORD=root_password # 这里配置root密码
     - MYSQL_DATABASE=nextcloud
     - MYSQL_USER=user_name # 这里配置一个非root账户给nextcloud使用
     - MYSQL_PASSWORD=user_password # 这里配置上面那个账号的密码
```



然而，新版本的`docker-compose`和**部分特定镜像**可能会读不到此时的环境变量，就会导致数据库容器在创建时没有设置密码，同时也无法知道`root`密码是什么，最终可能导致服务启动失败。



在查阅了多方`issue`之后，我把`environment`改成了键值对的形式：

```yml
environment:
      MYSQL_ROOT_PASSWORD: set_your_password
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: set_your_username
      MYSQL_PASSWORD: set_your_pass
```



#### 数据库启动

在搭建过程中遇到了`innodb-read-only`相关的`BUG`导致服务启动了但是无法安装，因此在启动服务时，我们直接利用`command`把`innodb-read-only`干掉，并解除内存限制：

```yml
command: [--transaction-isolation=READ-COMMITTED,--binlog-format=ROW,--innodb-file-per-table=1,--skip-innodb-read-only-compressed]
# to avoid innodb bugs
```



#### 完整的docker-compose

`docker-compose.yml`

```yml
version: '3.4'
services:

  db: 
    image: mariadb 
    restart: unless-stopped 
    expose: 
     - "3306"
    volumes:
     - ./db:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: set_your_password
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: set_your_username
      MYSQL_PASSWORD: set_your_pass
    command: [--transaction-isolation=READ-COMMITTED,--binlog-format=ROW,--innodb-file-per-table=1,--skip-innodb-read-only-compressed]
    # to avoid innodb bugs

  cache:
    image: redis
    restart: unless-stopped
    expose:
     - "6379"
    volumes:
     - ./cache:/data
    command: redis-server --requirepass 'redis_password'

  app:
    image: nextcloud:fpm
    restart: unless-stopped
    expose:
     - "9000"
    volumes:
     - ./app/html:/var/www/html
     - ./app/data:/var/www/html/data
     - ./app/config:/var/www/html/config
     - ./app/custom_apps:/var/www/html/custom_apps
    links:
     - db:db
     - cache:cache
    depends_on:
     - db
     - cache

  proxy:
    image: nginx
    restart: unless-stopped
    expose:
      - "80"
    ports:
     - 5000:443
    volumes:
     - ./app/html:/var/www/html
     - ./proxy/conf.d:/etc/nginx/conf.d:ro
     - ./proxy/ssl_certs:/etc/nginx/ssl_certs:ro
    links:
     - app:app
    depends_on:
     - app
```



#### 完整的nextcloud.conf

```nginx
upstream php-handler {
    server app:9000;
}

server {
    listen 80;
    listen [::]:80;
    server_name your.domain.com;
    # input your domain here
    # enforce https
    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your.domain.com;

    # Use Mozilla's guidelines for SSL/TLS settings
    # https://mozilla.github.io/server-side-tls/ssl-config-generator/
    # NOTE: some settings below might be redundant
    ssl_certificate /etc/nginx/ssl_certs/cert.cer;
    ssl_certificate_key /etc/nginx/ssl_certs/cert.key;

    # Add headers to serve security related headers
    # Before enabling Strict-Transport-Security headers please read into this
    # topic first.
    #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
    #
    # WARNING: Only add the preload option once you read about
    # the consequences in https://hstspreload.org/. This option
    # will add the domain to a hardcoded list that is shipped
    # in all major browsers and getting removed from this list
    # could take several months.
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;
    add_header Strict-Transport-Security  15552000;
    #add_header X-Frame-Options SAMEORIGIN;


    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Path to the root of your installation
    root /var/www/html;



    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;

    # The following rule is only needed for the Social app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/webfinger /public.php?service=webfinger last;

    location = /.well-known/carddav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    location / {
        rewrite ^ /index.php$request_uri;
    }

    location ~ ^\/(?:build|tests|config|lib|3rdparty|templates|data)\/ {
        deny all;
    }
    location ~ ^\/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+)\.php(?:$|\/) {
        fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        # Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        # Enable pretty urls
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^\/(?:updater|oc[ms]-provider)(?:$|\/) {
        try_files $uri/ =404;
        index index.php;
    }

    # Adding the cache control header for js, css and map files
    # Make sure it is BELOW the PHP block
    location ~ \.(?:css|js|woff2?|svg|gif|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
        #
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        add_header Referrer-Policy no-referrer;

        # Optional: Don't log access to assets
        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
        try_files $uri /index.php$request_uri;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```





## 故障排除

先确保容器正常运行：`docker-compose ps`查看容器状态：都是`Up`说明运行成功

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9tsjkc4kj30t306kgq5.jpg)



#### Nginx一直重启

##### 证书设置

如果出现`proxy`一直在重启的情况，可以查看一下日志`docker-compose logs`：大部分时候应该是没放证书或者证书设置错误......

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9tv2mypbj30jr080tdx.jpg)



比如这个就是典型的忘记设置证书了：重新配置证书即可

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9tvonhrrj30xj05q7cg.jpg)



##### 关闭https

如果没有证书的话，把`nextcloud.conf`设置成`http`的即可：

把`443`那一段直接复制粘贴到`80`上；同时修改`docker-compose.yml`的端口映射：

```nginx
## nextcloud.conf
upstream php-handler {
    server app:9000;
}

server {
    listen 80;
    listen [::]:80;

    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;
    add_header Strict-Transport-Security  15552000;
    #add_header X-Frame-Options SAMEORIGIN;


    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Path to the root of your installation
    root /var/www/html;



    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;

    # The following rule is only needed for the Social app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/webfinger /public.php?service=webfinger last;

    location = /.well-known/carddav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    location / {
        rewrite ^ /index.php$request_uri;
    }

    location ~ ^\/(?:build|tests|config|lib|3rdparty|templates|data)\/ {
        deny all;
    }
    location ~ ^\/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+)\.php(?:$|\/) {
        fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        # Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        # Enable pretty urls
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^\/(?:updater|oc[ms]-provider)(?:$|\/) {
        try_files $uri/ =404;
        index index.php;
    }

    # Adding the cache control header for js, css and map files
    # Make sure it is BELOW the PHP block
    location ~ \.(?:css|js|woff2?|svg|gif|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
        #
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        add_header Referrer-Policy no-referrer;

        # Optional: Don't log access to assets
        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
        try_files $uri /index.php$request_uri;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```



```yml
## docker-compose的proxy模块：把443改成80
proxy:
    image: nginx
    restart: unless-stopped
    expose:
      - "80"
    ports:
     - 5000:80
    volumes:
     - ./app/html:/var/www/html
     - ./proxy/conf.d:/etc/nginx/conf.d:ro
     - ./proxy/ssl_certs:/etc/nginx/ssl_certs:ro
    links:
     - app:app
    depends_on:
     - app
```



重新构建容器：

```bash
docker-compose down
# 关闭并删除原有容器

docker-compose up -d
# 构建新容器
```



此时我就部署了一个`http`服务的`nextcloud`：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9u7dpc1qj31hc0smazp.jpg)



#### SQLSTATE[HY000] [2002] No such file or directory

注册时出现数据库找不到`localhost`的情况：

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9uamxc8ij31hc0q61h0.jpg)



问题原因：

因为我们用的数据库的是容器，所以不能直接用主机的`localhost`或者`localhost:3306`去寻找主机。

解决方案：

因该通过容器的`DNS`服务寻找，比如在我们项目中，就应该输入`db:3306`

<img src="https://tva1.sinaimg.cn/large/0084b03xgy1gww6os69z3j30kg0jtjyx.jpg" alt="image-20211110102356732.png" style="zoom:67%;" />



#### 您的数据目录可被其他用户读取

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9uetq2zqj30or0a10xj.jpg)

问题原因：这里用的是`WSL`做演示。`WSL`用的是`Windows`的`NTFS`文件系统，权限是由`Windows`控制的，故所有目录都是`0770`......

解决方案：别用`WSL`......整一个纯正的`Linux`系统就行。



#### 其他数据库问题

* `innodb`相关的问题[前面](#环境变量)提到了，参考项目中`docker-compose.yml`重新设置即可。
* 环境变量的问题[前面](#数据库启动)也提到了，改成键值对的形式即可。
* ~~如果是直接克隆下来的项目，应该不会有这俩问题~~



## 最终效果

最后安装完毕大概长这样啦：这个是没怎么设置的原始`UI`

![image.png](https://tva1.sinaimg.cn/large/0084b03xly1gw9umtuspij31hc0q67c6.jpg)



## 小结

容器操作不难，编排也不难，难的是写出了配置文件然后疯狂`debug`......



## Refference

* https://hexo.chensmallx.top/2021/04/08/nextcloud-on-docker/

