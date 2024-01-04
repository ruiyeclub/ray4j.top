---
title: 开发必备软件-Typora配合PicGo阿里云图床配置(转)
date: 2024-01-04 11:20:40
tags:
---

> 写博客的时候，刚开始直接在各大平台上直接写，后来还是觉得不太方便，需要在各大平台之间来回切换。于是就改用Typora，但是有个问题就是图片的处理，只能放在本地。想要发布到各大平台，就需要图床。本文结合阿里云OSS、PicGo图床配置、Typora配置，能够做到一次文章编写，各大平台统一输出。

## 阿里云配置

### **创建对象存储Bucket**

登录阿里云后，选择对象存储

![image-20221222125850693](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/4b40339b-f3ee-415a-8348-1a371e5c4b6d.png)

新人开通试用3个月免费

![image-20221222130109938](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/14da8647-0f63-43ae-bcfa-69b5829db289.png)

开通后，进入管理控制台

点击Bucket列表，创建Bucket

![image-20221222130233633](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/dc718170-d70f-458c-918b-67d2c495739e.png)

访问控制开启公共读

![image-20221222130527365](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/5154a85d-5292-49b2-bd4d-035cd8fadc36.png)

### **访问控制RAM配置**

点击右上角AccessKey管理

![image-20221222134259058](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/10ecacba-67dd-4bd0-b2b0-05d6bd2c8280.png)

**创建AccessKey，注意这里的AccessKey ID 和 AccessKey Secret保存下来，后面要用到**

![image-20221222134507363](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/46ad3a39-cd1b-49b9-9d72-d4dc2e43730f.png)

## PicGo下载安装

下载地址

https://picgo.github.io/PicGo-Doc/zh/guide/#%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85

可以选择山东大学的镜像站

https://mirrors.sdu.edu.cn/github-release/Molunerfinn_PicGo

![image-20221222133605621](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/9bc5c527-4845-4ba7-b575-f6ac1c57ea3d.png)

### **PicGo配置**

- KeyId 、KeySecret 就是上面阿里云的AccessKey ID 和 AccessKey Secret
- Bucket 就是阿里云的bucket名称

![image-20221222134810396](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/326fb8c6-44a6-491b-96e8-c58bc4c08282.png)

- 存储区域，对应阿里云Endpoint的前面，我这里是oss-cn-hangzhou
- 存储路径，自己随便定义一个，比如 img/
- 自定义域名，对应阿里云 Bucket域名

![image-20221222135020880](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/6cf44a45-802d-4553-a499-c43726a92782.png)

## Typora下载及配置

Typora目前收费，想要免费可使用，可参考笔者的这个下载地址

> 链接：https://pan.baidu.com/s/1bCxmIPk23R0B9SVCqY37wg?pwd=F93i 提取码：F93i

### **设置**

找到设置，点击图像设置

上传图片的服务，选择PicGo.app

![image-20221222135334696](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/18f35e1f-8a91-4200-a71d-91b6d8f65ec9.png)

**注意：PicGo.app需要填写输入正确的安装路径，填写后可通过旁边的验证功能查看是否填写成功**



### **实际效果**

- 截图软件，粘贴效果，typora直接转换成了阿里云OSS的地址![image-20221222140236233](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/53d7a28b-96f3-4ba5-8fbc-e47b5bddae99.png)
- 外站粘贴过来的也是同样效果
