---
title: Spring Boot集成RabbitMQ消息队列
date: 2026-02-24 22:51:37
tags:
---

> 在两个Spring Boot项目中集成RabbitMQ，实现可靠的消息生产与消费。我们将使用Docker快速搭建RabbitMQ环境，并构建一个完整可用的消息队列系统。

## 一、快速搭建RabbitMQ环境

首先，我们使用Docker一键部署RabbitMQ服务：

```xml-dtd
docker run -d \
  -p 15672:15672 \
  -p 5672:5672 \
  --restart=always \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin \
  --name rabbitmq \
  --network=1panel-network \
  --hostname=rabbitmqhost \
  rabbitmq:management
```

**参数说明：**

- `-p 15672:15672`：管理界面端口
- `-p 5672:5672`：消息通信端口
- `--restart=always`：容器自动重启
- `-e`：环境变量，设置默认账号密码
- `--network`：指定网络（使用1panel-network）
- `rabbitmq:management`：带Web管理界面的版本

启动后访问：[http://localhost:15672](http://localhost:15672/)，使用admin/admin登录即可看到管理界面。

## 二、项目依赖配置

### 1. 添加Maven依赖

在两个Spring Boot项目的`pom.xml`中添加：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2. 统一配置文件

在两个项目的`application.yml`中配置RabbitMQ连接：

```
spring:
  rabbitmq:
    host: localhost          # RabbitMQ服务器地址
    port: 5672               # 消息通信端口
    username: admin          # 管理界面设置的用户名
    password: admin          # 密码
    virtual-host: /          # 虚拟主机
    # 生产者确认配置
    publisher-confirm-type: correlated
    publisher-returns: true
    
    listener:
      simple:
        acknowledge-mode: auto  # 自动确认
        prefetch: 10            # 每次预取消息数
        concurrency: 3          # 最小并发消费者
        max-concurrency: 10     # 最大并发消费者
```

## 三、生产者服务设计

### 1. RabbitMQ配置类

```java
@Configuration
public class RabbitMQConfig {
    
    // 交换机和队列常量定义
    public static final String DEMO_EXCHANGE = "demo.direct.exchange";
    public static final String DEMO_QUEUE = "demo.message.queue";
    public static final String DEMO_ROUTING_KEY = "demo.routing.key";
    
    // 创建直连交换机
    @Bean
    public DirectExchange demoExchange() {
        return ExchangeBuilder
                .directExchange(DEMO_EXCHANGE)
                .durable(true)      // 持久化
                .build();
    }
    
    // 创建消息队列
    @Bean
    public Queue demoQueue() {
        return QueueBuilder
                .durable(DEMO_QUEUE)
                .withArgument("x-dead-letter-exchange", "")  // 死信交换机
                .withArgument("x-dead-letter-routing-key", "demo.dlq")
                .build();
    }
    
    // 绑定队列到交换机
    @Bean
    public Binding demoBinding() {
        return BindingBuilder
                .bind(demoQueue())
                .to(demoExchange())
                .with(DEMO_ROUTING_KEY);
    }
    
    // JSON消息转换器
    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

### 2. 生产者服务实现

```java
@Service
@Slf4j
public class MessageProducerService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @Autowired
    private RabbitMQConfig rabbitMQConfig;
    
    /**
     * 发送简单文本消息
     */
    public boolean sendTextMessage(String content) {
        try {
            rabbitTemplate.convertAndSend(
                rabbitMQConfig.DEMO_EXCHANGE,
                rabbitMQConfig.DEMO_ROUTING_KEY,
                content
            );
            log.info("消息发送成功: {}", content);
            return true;
        } catch (Exception e) {
            log.error("消息发送失败: {}", content, e);
            return false;
        }
    }
    
    /**
     * 发送对象消息（自动JSON序列化）
     */
    public void sendObjectMessage(MessageDTO messageDTO) {
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId(UUID.randomUUID().toString());
        
        rabbitTemplate.convertAndSend(
            rabbitMQConfig.DEMO_EXCHANGE,
            rabbitMQConfig.DEMO_ROUTING_KEY,
            messageDTO,
            message -> {
                // 设置消息属性
                message.getMessageProperties().setMessageId(correlationData.getId());
                message.getMessageProperties().setTimestamp(new Date());
                message.getMessageProperties().setContentType("application/json");
                return message;
            },
            correlationData
        );
        
        log.info("对象消息发送完成，消息ID: {}", correlationData.getId());
    }
    
    /**
     * 带有确认回调的消息发送
     */
    @PostConstruct
    public void initConfirmCallback() {
        // 消息发送到Exchange确认回调
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("消息到达Exchange成功，消息ID: {}", correlationData.getId());
            } else {
                log.error("消息到达Exchange失败，消息ID: {}，原因: {}", 
                         correlationData.getId(), cause);
            }
        });
        
        // 消息路由到Queue失败回调
        rabbitTemplate.setReturnsCallback(returned -> {
            log.error("消息路由到队列失败: {}", returned.getMessage());
        });
    }
}
```

### 3. REST API接口

```
@RestController
@RequestMapping("/api/message")
@Slf4j
public class MessageController {
    
    @Autowired
    private MessageProducerService producerService;
    
    @PostMapping("/send")
    public ResponseEntity<?> sendMessage(@RequestBody MessageRequest request) {
        boolean result = producerService.sendTextMessage(request.getContent());
        
        if (result) {
            return ResponseEntity.ok(
                ResponseResult.success("消息发送成功")
            );
        } else {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ResponseResult.error("消息发送失败"));
        }
    }
    
    @PostMapping("/send-object")
    public ResponseEntity<?> sendObjectMessage(@RequestBody MessageDTO messageDTO) {
        try {
            producerService.sendObjectMessage(messageDTO);
            return ResponseEntity.ok(
                ResponseResult.success("对象消息发送成功")
            );
        } catch (Exception e) {
            log.error("发送对象消息异常", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ResponseResult.error("对象消息发送失败"));
        }
    }
}
```

## 四、消费者服务设计

### 1. 消费者服务实现

```java
@Component
@Slf4j
public class MessageConsumerService {
    
    /**
     * 监听队列消息 - 自动确认模式
     */
    @RabbitListener(
        queues = RabbitMQConfig.DEMO_QUEUE,
        concurrency = "3-10"  // 动态调整消费者数量
    )
    public void handleMessage(String message) {
        log.info("消费者[{}]收到消息: {}", 
                Thread.currentThread().getName(), message);
        
        try {
            // 模拟业务处理
            processBusinessLogic(message);
            log.info("消息处理完成: {}", message);
        } catch (Exception e) {
            log.error("处理消息失败: {}", message, e);
            // 异常时根据策略决定是否重新入队
            throw new AmqpRejectAndDontRequeueException("处理失败，不重新入队");
        }
    }
    
    /**
     * 监听对象消息
     */
    @RabbitListener(queues = RabbitMQConfig.DEMO_QUEUE)
    public void handleObjectMessage(MessageDTO messageDTO) {
        log.info("收到对象消息: ID={}, Content={}", 
                messageDTO.getId(), messageDTO.getContent());
        
        // 处理对象消息
        // ...
    }
    
    /**
     * 手动ACK模式 - 更精确的控制
     */
    @RabbitListener(queues = RabbitMQConfig.DEMO_QUEUE)
    public void handleMessageWithAck(
            Message message, 
            Channel channel) throws IOException {
        
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        
        try {
            String msgBody = new String(message.getBody());
            log.info("处理消息（手动ACK）: {}", msgBody);
            
            // 业务处理
            boolean success = processBusinessLogic(msgBody);
            
            if (success) {
                // 确认消息
                channel.basicAck(deliveryTag, false);
                log.info("消息确认成功: {}", deliveryTag);
            } else {
                // 拒绝消息，不重新入队
                channel.basicNack(deliveryTag, false, false);
                log.warn("消息处理失败，已拒绝: {}", deliveryTag);
            }
            
        } catch (Exception e) {
            log.error("处理消息异常，消息ID: {}", deliveryTag, e);
            // 异常时拒绝消息
            channel.basicNack(deliveryTag, false, false);
        }
    }
    
    private void processBusinessLogic(String message) {
        // 实际业务逻辑处理
        // 这里可以调用其他Service进行业务处理
        log.debug("处理业务逻辑: {}", message);
        
        // 模拟处理耗时
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### 2. 死信队列配置（处理失败消息）

```
@Configuration
public class DeadLetterConfig {
    
    public static final String DLX_EXCHANGE = "demo.dlx.exchange";
    public static final String DLQ_QUEUE = "demo.dlq.queue";
    public static final String DLQ_ROUTING_KEY = "demo.dlq";
    
    // 死信交换机
    @Bean
    public DirectExchange dlxExchange() {
        return new DirectExchange(DLX_EXCHANGE);
    }
    
    // 死信队列
    @Bean
    public Queue dlqQueue() {
        return QueueBuilder.durable(DLQ_QUEUE).build();
    }
    
    // 绑定死信队列
    @Bean
    public Binding dlqBinding() {
        return BindingBuilder.bind(dlqQueue())
                .to(dlxExchange())
                .with(DLQ_ROUTING_KEY);
    }
}
```

### 3. 死信队列消费者

```java
@Component
@Slf4j
public class DeadLetterConsumer {
    
    @RabbitListener(queues = DeadLetterConfig.DLQ_QUEUE)
    public void handleDeadLetter(Message failedMessage) {
        log.error("收到死信消息: {}", new String(failedMessage.getBody()));
        log.error("原始路由Key: {}", failedMessage.getMessageProperties().getReceivedRoutingKey());
        log.error("失败原因: {}", failedMessage.getMessageProperties().getHeader("x-death"));
        
        // 这里可以实现死信消息的处理逻辑
        // 1. 记录到数据库
        // 2. 发送告警通知
        // 3. 人工干预处理
    }
}
```

## 五、业务实体定义

### 消息传输对象

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class MessageDTO implements Serializable {
    
    private String id;
    private String content;
    private String messageType;
    private Date createTime;
    private Map<String, Object> extraParams;
    
    public MessageDTO(String content) {
        this.id = UUID.randomUUID().toString();
        this.content = content;
        this.createTime = new Date();
        this.extraParams = new HashMap<>();
    }
}
```

### API请求响应对象

```java
@Data
public class MessageRequest {
    private String content;
    private String type;
}

@Data
public class ResponseResult<T> {
    private Integer code;
    private String message;
    private T data;
    
    public static <T> ResponseResult<T> success(T data) {
        ResponseResult<T> result = new ResponseResult<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        return result;
    }
    
    public static ResponseResult<?> success(String message) {
        ResponseResult<?> result = new ResponseResult<>();
        result.setCode(200);
        result.setMessage(message);
        return result;
    }
}
```

## 六、测试验证

### 1. 发送消息测试

使用Postman或curl测试生产者API：

```
# 发送文本消息
curl -X POST http://localhost:8080/api/message/send \
  -H "Content-Type: application/json" \
  -d '{"content":"Hello RabbitMQ"}'

# 发送对象消息
curl -X POST http://localhost:8080/api/message/send-object \
  -H "Content-Type: application/json" \
  -d '{"content":"测试消息","messageType":"ORDER","extraParams":{"orderId":"123456"}}'
```

### 2. 监控管理

访问RabbitMQ管理界面：[http://localhost:15672](http://localhost:15672/)

在这里可以：

- 查看队列消息堆积情况
- 监控消费者连接
- 手动管理队列（清除、删除等）
- 查看消息流量统计

## 七、生产环境建议

### 1. 连接池配置

```
spring:
  rabbitmq:
    connection-timeout: 5000      # 连接超时时间
    cache:
      channel:
        size: 25                  # 通道缓存大小
      connection:
        mode: connection          # 连接模式
        size: 5                   # 连接池大小
```

### 2. 高可用配置

```
spring:
  rabbitmq:
    addresses: rabbit1:5672,rabbit2:5672,rabbit3:5672  # 集群地址
```

### 3. 消息可靠性保障

- **生产者确认**：确保消息到达Exchange
- **持久化**：队列、消息都设置为持久化
- **消费者ACK**：处理成功后才确认消息
- **死信队列**：处理失败的消息
- **消息幂等**：消费者端实现幂等处理

## 八、总结

通过本文的完整实现，我们构建了一个生产可用的RabbitMQ消息队列系统，具备以下特点：

1. **快速部署**：Docker一键部署RabbitMQ
2. **完整集成**：Spring Boot完美整合
3. **可靠传输**：支持生产者确认、消息持久化
4. **灵活消费**：支持自动/手动ACK、并发控制
5. **容错处理**：死信队列机制
6. **易于监控**：Web管理界面实时监控

这个方案可以广泛应用于：

- 订单处理异步化
- 日志收集与分析
- 系统解耦
- 流量削峰填谷
- 数据同步等场景

希望这篇实战指南能帮助你在项目中顺利集成RabbitMQ！如果有任何问题，欢迎在评论区讨论。