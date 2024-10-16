---
title: Docker搭建和部署禅道
date: 2024-10-16 13:59:25
tags:
---

> 禅道项目管理工具提供了丰富的功能模块，包括任务管理、需求管理、Bug跟踪、版本控制、团队协作、项目报表等，这些功能覆盖了项目管理的各个方面。全面的功能模块使得项目团队可以在一个统一的平台上进行协同工作，无需再整合多个系统，从而提高了工作效率。

公司使用禅道来管理项目需求、进度以及Bug测试，新版禅道添加了很多功能，这里我搭建使用的是老版本12.5.3。

### Docker拉取禅道镜像

```shell
docker pull idoop/zentao:12.5.3
```

![image-20241016134837422](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20241016134837422.png)

### 运行禅道容器

```shell
docker run --privileged=true -d -p 8082:80 -p 3316:3306 \
        -e ADMINER_USER="admin" -e ADMINER_PASSWD="123456" \
        -v /dockerdatafile/chandao/:/opt/zbox/ \
        --name zentao-server \
        idoop/zentao:12.5.3
```

参数说明：

- ADMINER_USER：指定禅道管理员账号

- ADMINER_PASSWD：指定禅道管理员首次登录密码

### 访问禅道

浏览器访问宿主机器地址加8082端口，登录使用用户名 admin，密码 123456

![image-20241016135156614](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20241016135156614.png)

### 使用1Panel创建网站

可通过域名进行访问

![image-20241016135651301](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20241016135651301.png)

 