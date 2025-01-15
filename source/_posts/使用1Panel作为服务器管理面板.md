---
title: 使用1Panel作为服务器管理面板
date: 2025-01-15 14:22:14
tags:
---

> 📝记录一下使用1panel作为服务器管理面板，之前基本是使用手搓命令构建服务，了解过宝塔面板，感觉太过臃肿且需要注册账户，后续考虑服务部署都是用1panel面板进行操作。

官方文档：https://1panel.cn/docs/

## 一、产品介绍

1Panel 是一个现代化、开源的 Linux 服务器运维管理面板。
![UI展示](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/overview.png)

## 二、安装部署

1. Ubuntu安装：

```
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```

![image-20250115134821023](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20250115134821023.png)

2. 通过访问地址和面板用户密码进行登录

## 三、功能使用

1. 前端页面创建网址后选择静态网址进行配置，后台选择反向代理

![image-20250115140517787](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20250115140517787.png)

2. 通过网址设置配置HTTPS和设置NGINX

![图像_副本](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/%E5%9B%BE%E5%83%8F_%E5%89%AF%E6%9C%AC.png)

## 四、计划任务

可以通过linux定时任务自动执行shell等脚本命令

![截屏2025-01-15 14.16.51](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/%E6%88%AA%E5%B1%8F2025-01-15%2014.16.51.png)
