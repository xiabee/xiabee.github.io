---
title: Docker搭建Wordpress个人博客
date: 2021-11-9 20:00:23
tags:
  - docker
  - wordpress
categories:
  - coding
abbrlink: wordpress-docker
---



## Wordpress简介

* [官网](https://wordpress.org/)
* 适合新手入门的博客框架，常用于个人博客的搭建，正如其名字一样，通过简单的几个`word`即可搭建出一个博客



## 太长不看版

* 项目地址：[Github](https://github.com/xiabee/wordpress-docker)
* 功能：通过`docker-compose`，实现了利用容器搭建`nginx`+`mariadb`+`wordpress`的组合，在VPS中搭建个人博客
* 运行方式：详情见项目的`README.md`
* 环境依赖：`docker`

```bash
sudo apt install docker docker-compose
git clone https://github.com/xiabee/wordpress-docker
cd wordpress-docker
```



### 搭建HTTP服务 

```bash
rm ./nginx/nginx_https.conf
# 删除https
docker-compose up -d
# 构建http服务
```



### 搭建HTTPS服务

如果要搭建`https`服务的话需要在`nginx_https.conf`中配置一下证书路径，并删除`nginx.conf`，重新构建服务。

```bash
rm ./nginx/nginx.conf
docker-compose up -d
```





## 代码详解

### docker-compose.yml

* [官方文档](https://docs.docker.com/compose/)

* 简言之：调用`Docker`服务的`API`进行容器管理，定义和运行多个`Docker`容器应用。

`network`模块这里直接跳过，就是定义一下网络类型和网络ID，这里不需要过多设置，我们直接跳到服务`services`模块



#### wordpress模块

```yml
wordpress:
        # 选中带有php-fpm 的版本
        image: wordpress:5.8.1-php8.0-fpm
        # 把wordpress的主体文件夹映射到本地 wordpress目录
        volumes:
            - ./wordpress:/var/www/html
        
        # 环境变量 设置密码
        environment:
            WORDPRESS_DB_HOST: mariadb:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress-pass
            WORDPRESS_DB_NAME: wordpress
        # 设置依赖
        depends_on:
            - mariadb
        restart: always
        networks:
            - xiabee
```

* `image`：这里选择带有`FPM`的`wordpress`镜像，因为要配合`nginx`使用
* `volumes`：建立目录映射，将容器目录映射到宿主机中，方便我们管理和维护容器的文件
* `environment`：设置环境变量，用于设置数据库的密码
* `depends_on`：设置依赖，这里`wordpress`需要依赖`mariadb`数据库启动
* `restart`：设置重启方式，这里选择`always`，即挂掉就重启。

* `networks`：这里直接用了前面定义的网络



#### mariadb

```yml
mariadb:
        image: mariadb:latest
        expose:
          - "3306"
        volumes:
          - ./mysql:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: root-pass
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress-pass
            MYSQL_RANDOM_ROOT_PASSWORD: 1
          # 使用随机root密码

        #这里使用命令登陆，删除后可能于新版mysql不兼容
        command: [--default-authentication-plugin=mysql_native_password, --character-set-server=utf8mb4, --collation-server=utf8mb4_unicode_ci]
        #挂掉自动重启
        restart: always
        networks:
            - xiabee
```

* `command`：使用命令方式登录，进行设置，避免`BUG`
* 其他与`wordpress`的操作基本相同



#### nginx

```yml
nginx:
        image: nginx:latest
        ports:
            - '80:80'
            - '443:443'
        # 映射本地，加载本地的配置
        volumes:
            - ./nginx:/etc/nginx/conf.d    
            #注意服务器配置文件的映射位置，如果需要修改配置，直接修改映射的文件即可
            - ./logs/nginx:/var/log/nginx
            - ./wordpress:/var/www/html  
            #这里选择本地wordpress即 wordpress。docker中的目录
        depends_on:
            - wordpress
        restart: always
        networks:
            - xiabee
```

* 这里注意目录映射的位置就行，其他和`wordpress`的设置基本相同



### nginx.conf

* 这个文件用来配置`http`服务
* 开启`http`服务时需要删除`nginx_https.conf`

```nginx
server {
listen 80;              #设置监听端口，http默认端口为80
server_name localhost;

    root /var/www/html;
    index index.php;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

* `listen`：监听端口
* `server_name`：设置你的域名
* `root`：根目录地址
* `location`：设置访问的`url`，这里设置的是根目录的访问，即访问网站根目录时，跳转到`/index.php`上
* 第二个`location`用于监听9000端口的`wordpress`，对`wordpress`进行反向代理



### nginx_https.conf

* 这个文件用于构建`https`服务

* 开启`https`需要提前拥有域名和证书
* 开启`https`服务时需要删除`nginx.conf`



#### 禁止IP直接访问

```nginx
server {
    listen  80 default;
    server_name ~.*;
    return 500;
}   #禁止直接ip访问
```

这里创建一个`server`用于禁止IP直接访问



#### 强制HTTPS

```nginx
server {
  listen   80;
  server_name  domain name.com;     #这里的domain name.com换成您的域名
  return   301 https://$server_name$request_uri;
}   #强制全站使用https
```

这里的`server`用于强制全站使用`https`



#### 设置证书文件

```nginx
server {
    listen 443 ssl;
    server_name domain name.com; 
    #这里的domain name.com换成您的域名

    root /var/www/html;
    index index.php;

    ssl_certificate ./conf.d/domain name.com;  
    #将domain name.pem替换成您的证书文件
    ssl_certificate_key ./conf.d/domain name.key;   
    #将domain name.key替换成您的密钥文件
    
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  
    #使用此加密套件。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   
    #使用该协议进行配置。
    ssl_prefer_server_ciphers on;   

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}   #https服务器
```



这里需要把自己的证书添加到宿主机的`nginx`目录下，因为前面做了目录映射，所以证书会自动映射到容器的`conf.d`目录中



## 常见BUG

### /docker/api/client.py错误

* 容器服务没开......开启容器服务，重新运行

  ![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gwdkucofnzj30u207279r.jpg)



### 数据库错误

* 在`docker-compose`升级之后，`environment`的设置必须像仓库中那样以键值对的形式存在，不能直接输入一个字符串，否则会读不到环境变量，导致数据库密码设置错误，最终导致连不上服务器

* 如果你是直接CV这个项目的话应该不会有问题，除非你数据库密码填错了（x）



### nginx一直重启

* 检查一下是不是配置了`https`但是证书没有复制到目录内......

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gwdkyxpyfsj31150rehdt.jpg)

* 如果不想设置`https`，那就直接把`nginx_https.conf`删掉，重启容器就行

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gwdl0emfwwj314n0d77fp.jpg)



## 最终安装

如果所有容器都正常启动了，直接访问你的`IP/域名`应该就能看到安装界面了（`https`服务如果没有做强制跳转，则需访问你域名的`443`端口才能访问）

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gwdl0zor35j31890ql79w.jpg)



安装完成之后，剩下的事情对着说明做就行了，~~如果实在看不懂就装个中文版的~~......



## 关于主题

* [xiabee.cn](https://xiabee.cn/)用的是[樱花庄](https://github.com/mashirozx/Sakura)主题：个人感觉还算好看，就是不够简洁，所以转战`hexo`了哈哈哈（x）

![image.png](https://s3.xiabee.cn/pic/weibo-backup/0084b03xly1gwpgbyy55pj31hc0smkai.jpg)



* 不过之后可能会再找几个好看的主题
