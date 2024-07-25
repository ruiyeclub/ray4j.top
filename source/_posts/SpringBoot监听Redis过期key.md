---
title: SpringBoot监听Redis过期key
date: 2024-07-25 19:22:22
tags:
---

> 在SpringBoot项目中配置Redis的过期提醒，通过修改`redis.conf`或命令行设置`notify-keyspace-events`，并创建自定义的Redis消息监听器来处理过期事件。



### 一、开启Redis key过期提醒

- 方式一：修改redis.conf配置文件

```shell
# 默认 notify-keyspace-events ""
notify-keyspace-events Ex
```

- 方式二：命令行开启

```shell
CONFIG SET notify-keyspace-events Ex
CONFIG GET notify-keyspace-events
```



### 二、项目导入redis依赖

```shell
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
 </dependency>
```



### 三、配置config配置类

```java
@Configuration
public class RedisListenerConfig {

	@Bean
	RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory) {
		RedisMessageListenerContainer container = new RedisMessageListenerContainer();
		container.setConnectionFactory(connectionFactory);
		return container;
	}
}
```



### 四、配置监听代码

```java
@Slf4j
@Component
public class KeyExpiredListener extends KeyExpirationEventMessageListener {

    public KeyExpiredListener(RedisMessageListenerContainer listenerContainer) {
        super(listenerContainer);
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String channel = new String(message.getChannel(), StandardCharsets.UTF_8);
        // 过期的key
        String key = new String(message.getBody(), StandardCharsets.UTF_8);
        log.info("redis key 过期：pattern={},channel={},key={}", new String(pattern), channel, key);
        if (key.equals(INVEST_WS_EXPIRED_KEY)) {
            // 处理业务逻辑...
        }
    }
}
```