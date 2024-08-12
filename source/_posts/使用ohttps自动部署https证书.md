---
title: 使用ohttps自动部署https证书
date: 2024-08-12 13:03:01
tags:
---

> 在使用第三方托管网站做个人网站，例如：Github Pages，想要给网站加上https，可以通过ohttps来创建Let's Encrypt的https证书，单纯想给网站加上https也可以。


### 安装Nginx

先确保服务器(我这里是win服务器)已经安装了[nginx](https://nginx.org/en/download.html)，选择适合自己的版本，然后下载解压到服务器上，双击运行nginx.exe，访问服务器ip地址，出现Welcome to Nginx!则表示安装启动成功。



### 使用ohttps部署ssl证书

1. 创建证书

![image-20240812124550577](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240812124550577.png)

2. 需要在自己域名管理解析中加入ohttps的记录

![image-20240812124657919](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240812124657919.png)

3. 在nginx配置中需要ssl_certificate_key和ssl_certificate

**ssl_certificate_key**: 私钥文件通常包含与SSL证书配对的私钥，用于加密和解密通信中的数据，文件后缀.key，内容开头-----BEGIN RSA PRIVATE KEY-----

**ssl_certificate**: SSL证书文件包含用于验证服务器身份的公钥以及相关信息，如服务器的域名和证书颁发机构（CA）的签名，文件后缀为.crt或者是.pem，内容开头-----BEGIN CERTIFICATE-----

![image-20240812125505357](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240812125505357.png)

下载这两个文件，然后存放到服务器上

### 配置Nginx.conf

```
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    server_names_hash_bucket_size 64;
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen 443 ssl;
        server_name wechat.ruiyeclub.cn;

        ssl_certificate      绝对路径/key/fullchain.pem;
        ssl_certificate_key  路径/key/cert.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_pass http://localhost:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 80;
        server_name wechat.ruiyeclub.cn;

        ssl_certificate      绝对路径/key/fullchain.pem;
        ssl_certificate_key  路径/key/cert.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_pass http://localhost:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Nginx常用命令

- **启动Nginx服务** nginx

- **停止Nginx服务** nginx -s stop
- **检测配置文件的正确性** nginx -t
- **重新加载Nginx配置文件** nginx -s reload
- **查看Nginx版本信息** nginx -v
- **平滑停止Nginx服务** nginx -s quit