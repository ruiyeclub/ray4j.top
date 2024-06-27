---
title: Linux安装nginx和配置https
date: 2024-06-27 21:33:35
tags:
---

### 1.安装nginx

如果你还没有安装Nginx，可以使用你的Linux发行版的包管理器来安装。以下是一些常见发行版的安装命令：

- 对于Debian/Ubuntu系统：

```bash
sudo apt update  
sudo apt install nginx
```

- 对于CentOS/RHEL系统（使用yum）：

```bash
sudo yum install nginx
```

### 2.启动nginx

- 安装完成后，启动Nginx服务：

  ```bash
  sudo systemctl start nginx
  ```

- 并且设置它开机自启：

  ```bash
  sudo systemctl enable nginx
  ```

### 3. 检查Nginx状态

你可以检查Nginx的状态来确保它正在运行：

```bash
sudo systemctl status nginx
```

### 4. 配置Nginx

Nginx的主要配置文件通常位于`/etc/nginx/`目录下，特别是`nginx.conf`文件。但大多数配置更改通常会在`sites-available/`或`conf.d/`目录中的单独文件中进行，然后通过在`nginx.conf`中包含这些文件来应用。

- 打开Nginx的主配置文件（如果你需要直接编辑它）：

```bash
sudo nano /etc/nginx/nginx.conf
```

- 重新加载Nginx配置以使更改生效：

```bash
sudo systemctl reload nginx
```

或者你可以完全重启Nginx服务：

```bash
sudo systemctl restart nginx
```



## 二、Let's Encrypt在Linux上免费配置HTTPS

Let's Encrypt简介Let's Encrypt是一个由Mozilla、Cisco、Akamai、IdenTrust和EFF等组织发起的项目，旨在通过自动化方式为网站提供免费的SSL/TLS证书。它的目标是促进互联网的加密化，保障用户的在线安全。Let's Encrypt提供的证书有效期为90天，但可以通过自动化过程免费续期。

### 1.安装Certbot

```bash
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx
```

### 2.配置HTTPS

```bash
sudo certbot --nginx
```

![image-20240627212626690](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240627212626690.png)

在浏览器地址栏中输入你的域名并加上https://前缀（例如：https://www.example.com），如果浏览器显示了一个绿色的锁形图标，并且URL地址以https://开头，那么恭喜你，你的网站已经成功配置了HTTPS！

## 三、配置nginx.conf

```python
server {
		listen 443 ssl;
		server_name 域名.com;

		ssl_certificate /etc/letsencrypt/live/地址/fullchain.pem;
		ssl_certificate_key /etc/letsencrypt/live/地址/privkey.pem;

		location / {
			proxy_pass http://ip:port;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
		}
}
```

参考文章：[Let's Encrypt在Linux上免费配置HTTPS](https://zhuanlan.zhihu.com/p/696518542)