---
title: 🚀 Tomcat 对比 Undertow：高性能 Web 服务器的选择
date: 2025-12-08 16:45:11
tags:
---

这是一篇关于 **Tomcat 与 Undertow 对比**的文档，并附带了 **Undertow 的使用教程**。

-----

## 🚀 Tomcat 对比 Undertow：高性能 Web 服务器的选择

Apache Tomcat 和 Undertow 都是流行的 **Java Servlet 容器**和 **Web 服务器**，它们在 Java 生态系统中扮演着关键角色。然而，它们在设计哲学、性能和功能集上有所不同。

### 主要特点对比

| 特性 | Apache Tomcat | Undertow | 优势/特点 |
| :--- | :--- | :--- | :--- |
| **项目成熟度** | 非常高 | 相对较新（由 JBoss/Red Hat 开发） | **Tomcat** 更加成熟、稳定，社区支持广泛。 |
| **设计核心** | 基于 **传统线程模型** 和 **I/O 模型** | 基于 **NIO (Non-blocking I/O)** 和 **事件驱动** | **Undertow** 在高并发场景下通常有更好的性能和更低的资源消耗。 |
| **资源占用** | 相对较高（尤其在高并发下） | 相对较低 | **Undertow** 设计精简，启动快，内存占用少。 |
| **模块化** | 模块化程度适中 | 高度模块化，设计灵活 | **Undertow** 可以作为独立的 Web 服务器或嵌入到应用程序中，灵活性更高。 |
| **嵌入式使用** | 良好 | 优秀 | **Undertow** 在 Spring Boot 或其他微服务框架中作为**嵌入式容器**表现突出。 |
| **WebSockets 支持** | 良好 | 原生，高性能 | **Undertow** 的设计使其在处理 WebSockets 和 HTTP/2 等新协议时表现出色。 |
| **稳定性/可靠性** | 极高 | 优秀 | **Tomcat** 经过多年生产环境验证，可靠性是其最大的优势之一。 |

### 🔍 总结与选择建议

  * **选择 Tomcat:** 如果你的应用需要**极高的稳定性**、**广泛的社区支持**，或者你的环境是传统的 Java EE/Servlet 部署模式，并且对极高并发要求不苛刻，**Tomcat** 是一个安全且可靠的选择。
  * **选择 Undertow:** 如果你的应用是一个**微服务**或 **Spring Boot** 应用，追求**极致的性能**、**低资源消耗**，以及**高并发处理能力**，特别是需要充分利用 **NIO/事件驱动** 的优势时，**Undertow** 是一个优秀的、现代化的替代品。

-----

## 🛠️ Undertow 使用教程：嵌入式部署（Maven + Spring Boot）

Undertow 最大的优点之一是它作为 **嵌入式容器** 的卓越性能和简易性。在现代 Java 开发中，尤其是在使用 **Spring Boot** 时，切换到 Undertow 非常简单。

### 1\. 创建 Maven 项目

确保你的项目是一个标准的 **Spring Boot** 应用，并使用 **Maven** 或 **Gradle** 进行构建。

### 2\. 移除默认的 Tomcat 依赖

在 Spring Boot 的 **`pom.xml`** 文件中，默认的 `spring-boot-starter-web` 会引入 Tomcat。你需要**排除**它，然后显式地引入 Undertow。

#### `pom.xml` 配置 (Maven)

找到 `spring-boot-starter-web` 依赖，并添加 `<exclusions>` 标签来排除 Tomcat：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 3\. 引入 Undertow 依赖

在 `pom.xml` 中添加 `spring-boot-starter-undertow` 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

### 4\. 运行应用

完成上述步骤后，你的 Spring Boot 应用在启动时就会自动检测并使用 **Undertow** 作为其内置的 Web 服务器，而不是默认的 Tomcat。

### 5\. 配置文件进阶（可选）

你可以通过 Spring Boot 的 `application.properties` 或 `application.yml` 文件来配置 Undertow 的线程池大小等参数，以进行性能调优。

#### `application.properties` 示例

```properties
# 设定 Undertow 的 I/O 线程数 (通常等于CPU核心数)
server.undertow.threads.io=4

# 设定工作线程数 (处理请求的线程)
server.undertow.threads.worker=200

# 设定缓冲区大小
server.undertow.buffer-size=1024
```

通过以上简单的配置，你就可以在 Spring Boot 应用中轻松地从 Tomcat 切换到高性能的 Undertow 服务器。

-----

PS: 该内容由Gemini生成
