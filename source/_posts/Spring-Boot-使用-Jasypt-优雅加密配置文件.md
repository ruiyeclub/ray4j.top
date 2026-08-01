---
title: Spring Boot 使用 Jasypt 优雅加密配置文件
date: 2026-08-01 11:42:48
tags:
---

# Spring Boot 使用 Jasypt 优雅加密配置文件

> 本文适用于 Spring Boot 2.x / 3.x，演示如何对数据库密码、短信密钥、邮箱密码等敏感配置进行加密，避免配置文件裸奔。

------

## 一、为什么要用 Jasypt？

在实际项目中，`application.yml` / `application.properties` 里经常能看到这样的配置：

```
spring:
  datasource:
    password: 123456
```

一旦代码被误提交到 GitHub，数据库就直接“送人”了。

**Jasypt（Java Simplified Encryption）** 可以帮我们把这些敏感信息加密成密文，Spring Boot 启动时自动解密，既安全又无侵入。

------

## 二、依赖引入

```yml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.5</version>
</dependency>
```

> ✅ 推荐版本：`3.0.5`（稳定、兼容性好）

------

## 三、Jasypt 基础配置

### application.yml

```yml
jasypt:
  encryptor:
    password: ${JASYPT_PASSWORD:stock-jasypt-default}
    algorithm: PBEWithMD5AndDES
    salt-generator-classname: org.jasypt.salt.RandomSaltGenerator
    iv-generator-classname: org.jasypt.iv.NoIvGenerator
    property:
      prefix: ENC(
      suffix: )
```

### 🔑 密钥加载优先级（非常重要）

1. JVM 启动参数：`-Djasypt.encryptor.password=xxx`
2. 环境变量：`JASYPT_PASSWORD`
3. 默认值：`stock-jasypt-default`（⚠️ 仅限本地开发）

------

## 四、加密工具类（核心）

> 可直接复制使用

```java
import org.jasypt.encryption.pbe.StandardPBEStringEncryptor;
import org.jasypt.encryption.pbe.config.EnvironmentPBEConfig;

/**
 * Jasypt 加密工具类 —— 用于生成 ENC(密文)
 */
public class JasyptEncryptUtil {

    public static void main(String[] args) {
        String password = args.length > 1 ? args[1] : "stock-jasypt-default";
        String plainText = args.length > 0 ? args[0] : "PMYR2PHYSEWTfKBP";

        StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
        EnvironmentPBEConfig config = new EnvironmentPBEConfig();

        config.setAlgorithm("PBEWithMD5AndDES");
        config.setPassword(password);
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
        config.setIvGeneratorClassName("org.jasypt.iv.NoIvGenerator");

        encryptor.setConfig(config);

        String encrypted = encryptor.encrypt(plainText);
        String decrypted = encryptor.decrypt(encrypted);

        System.out.println("===== Jasypt 加密工具 =====");
        System.out.println("原始明文: " + plainText);
        System.out.println("加密结果: ENC(" + encrypted + ")");
        System.out.println("解密验证: " + decrypted);
        System.out.println("==========================");
    }
}
```

------

## 五、如何生成密文？

1. 打开 `JasyptEncryptUtil`
2. 修改 `plainText`
3. 右键 → Run

------

```
加密结果: ENC(A9xJ3ZkqW+2yQZ7vNc6tLw==)
```

------

## 六、配置文件中如何使用？

### YAML 示例

```yml
spring:
  datasource:
    password: ENC(A9xJ3ZkqW+2yQZ7vNc6tLw==)
```

### Properties 示例

```
mail.smtp.password=ENC(A9xJ3ZkqW+2yQZ7vNc6tLw==)
```

✅ Spring Boot 启动时会自动解密，**业务代码零改动**。

------

## 七、生产环境启动方式（重点）

⚠️ **千万不要把密钥写死在配置文件里！**

### 1️⃣ JVM 参数方式（推荐）

```
java -jar app.jar -Djasypt.encryptor.password=生产密钥
```

### 2️⃣ 环境变量方式

```
export JASYPT_PASSWORD=生产密钥
java -jar app.jar
```

### 3️⃣ Docker 方式

```
docker run -e JASYPT_PASSWORD=生产密钥 stock-app:latest
```

------

## 八、常见需要加密的配置

| 类型   | 配置项示例                   |
| ------ | ---------------------------- |
| 数据库 | `spring.datasource.password` |
| Redis  | `spring.redis.password`      |
| 邮箱   | `mail.smtp.password`         |
| 短信   | `sms.config.secret`          |
| OSS    | `oss.accessKeySecret`        |
| 支付   | `pay.merchant.key`           |

------

## 九、常见问题 & 避坑指南

### ❓ 为什么每次加密结果不一样？

✅ 正常现象！

使用了 `RandomSaltGenerator`，安全性更高。

------

### ❓ 启动报错：Bad credentials？

- 加密和解密的 **密钥不一致**
- 生产环境未正确传入 `JASYPT_PASSWORD`