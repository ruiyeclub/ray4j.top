---
title: Docker安装nacos使用mysql
date: 2025-05-08 14:03:23
tags:
---

要在 Docker 中安装并运行 Nacos 服务器并使用 MySQL 作为其数据库，您可以遵循以下步骤。Nacos 是一个更易于构建云原生应用的动态服务发现、配置和服务管理平台，它支持多种数据库作为其存储后端，包括 MySQL。

### 步骤 1: 安装 Docker

确保您的系统上已安装 Docker。您可以通过以下命令检查 Docker 是否已安装：

```
docker --version
```

如果未安装，请访问 [Docker 官网](https://docs.docker.com/get-docker/) 获取安装指南。

### 步骤 2: 启动 MySQL 容器

首先，您需要运行一个 MySQL 容器。可以使用官方的 MySQL 镜像。创建一个名为 `nacos-mysql` 的容器，并为它设置一些环境变量，例如 root 用户的密码和数据库名：

```
docker run --name nacos-mysql -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -d mysql:5.7
```

这条命令会启动一个 MySQL 5.7 容器，root 密码设置为 `root`，并且将容器的 3306 端口映射到宿主机的 3306 端口。

### 步骤 3: 创建 Nacos 所需的数据库和用户

登录到 MySQL 容器中，并创建一个数据库和用户供 Nacos 使用：

```
docker exec -it nacos-mysql mysql -uroot -p
```

输入密码后，执行以下 SQL 命令：

```
CREATE DATABASE nacos DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;CREATE USER 'nacos'@'%' IDENTIFIED BY 'nacos';GRANT ALL PRIVILEGES ON nacos.* TO 'nacos'@'%';FLUSH PRIVILEGES;EXIT;
```

### 步骤 4: 拉取并运行 Nacos 容器

接下来，拉取 Nacos 的官方 Docker 镜像并运行它，指定使用 MySQL 作为存储：

```
docker run --name nacos -e MODE=standalone -e MYSQL_SERVICE_HOST=nacos-mysql -e MYSQL_SERVICE_PORT=3306 -e MYSQL_SERVICE_DB_NAME=nacos -e MYSQL_SERVICE_USER=nacos -e MYSQL_SERVICE_PASSWORD=nacos -p 8848:8848 -d nacos/nacos-server:latest
```

这条命令会启动一个 Nacos 容器，配置为使用上面创建的 MySQL 数据库。`MYSQL_SERVICE_HOST` 应与 MySQL 容器的名称相同（这里是 `nacos-mysql`），而 `MYSQL_SERVICE_PORT` 是 MySQL 容器的端口（这里是 `3306`）。确保这些值与您在步骤 2 中设置的相匹配。

### 步骤 5: 验证 Nacos 是否正常运行

打开浏览器，访问 `http://localhost:8848/nacos`，默认用户名和密码都是 `nacos`。如果一切设置正确，您应该能够看到 Nacos 的管理界面。

通过以上步骤，您就可以在 Docker 中成功安装并运行 Nacos 服务器，使用 MySQL 作为其存储后端了。