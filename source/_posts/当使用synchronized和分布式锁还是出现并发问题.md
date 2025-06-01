---
title: 当使用synchronized和分布式锁还是出现并发问题
date: 2025-06-01 23:40:59
tags:
---

> 📝记录一下，尽管使用了同步锁甚至是分布式锁还是查询并发问题的情况。
>
>对同一条数据进行"查询状态->更新状态"操作时，出现多次更新状态。

这个是我业务代码：

```java

@Transactional(rollbackFor = Exception.class)
public void update(Long modelId) throws GseException {
    synchronized (this) {
        //业务逻辑代码
    }
}
```

使用redis分布式锁的代码：

```java

@Transactional(rollbackFor = Exception.class)
public void update(Long modelId) throws GseException {
    RLock lock = redissonClient.getLock(REDIS_COST_MODEL_ID_LOCK + modelId);
    try {
        if (lock.tryLock(20, 2, TimeUnit.SECONDS)) {
            //业务逻辑代码
        } else {
            log.error("获取分布式锁失败");
        }
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    } finally {
        // 判断当前线程是否持有锁
        if (lock.isHeldByCurrentThread()) {
            //释放当前锁
            lock.unlock();
        }
    }
}
```

### 一、存在的问题以及原因

问题就是以上两种写法都没有生效，但是为什么呢？ 在解释这个问题之前，我们首先要弄清楚两个问题：

1. @Transactional的底层实现原理，开启事务和提交事务的时机是什么？

2. 分布式锁，和本地锁机制释放锁的时机是什么时候？3.1 @Transactional的底层实现原理，开启事务和提交事务的时机是什么？

它的底层实现原理主要依赖于 Spring 的面向切面编程（AOP）机制。 底层实现原理

- AOP 代理：当一个类或方法被 @Transactional 注解标记时，Spring 容器在初始化 Bean 时会检测到这个注解。对于使用 Spring
  的代理模式（如 JDK 动态代理或 CGLIB），Spring 会为该 Bean
  创建一个代理对象。这个代理对象会在调用实际方法前后插入事务管理相关的代码，即在方法执行前开启事务，在方法执行完毕后根据执行情况提交或回滚事务。
- 解析注解：Spring 通过扫描 Bean 定义，识别出带有 @Transactional 注解的方法或类，并配置相应的事务属性，如传播行为、隔离级别、超时时间、是否只读等。
- 事务拦截器：Spring 使用 AOP 机制中的拦截器（Interceptor）或Advice（通常为 TransactionInterceptor 或 AspectJ
  的切面），在方法调用前后织入事务处理逻辑。在方法调用前，根据事务属性设置事务的开始；在方法正常结束时提交事务，如果方法抛出未检查异常（继承自
  RuntimeException 的异常）或已检查异常（被 @Transactional 的 rollbackFor 属性指定的异常）则回滚事务。

开启事务和提交事务的时机

- 开启事务：**事务通常在进入被 @Transactional 注解的方法之前立即开始**。这意味着在执行业务逻辑之前，Spring
  会确保与当前环境匹配的事务上下文已经建立。这包括选择合适的事务管理器，根据事务属性配置事务的隔离级别、传播行为等，并在数据库中实际开启事务。
- 提交事务：**如果被注解的方法正常执行结束，没有抛出任何异常，Spring 会在离开该方法之前提交事务**
  。提交事务意味着将所有挂起的更改永久化到数据库中，使事务中的所有操作对外可见。
- 回滚事务：如果在被注解的方法执行过程中抛出了异常，并且该异常未被 @Transactional 的 noRollbackFor 属性豁免，Spring
  将在捕获到异常后立即回滚事务，撤销所有在事务中已完成但未提交的操作，保持数据的一致性。

### 二、分布式锁，和本地锁机制释放锁的时机是什么时候？

答案是：本地锁，如果是synchronized，看你包裹起来的范围。Lock的话 看你手动释放锁的时候。 分布式锁：看你手动释放锁的时候。

那么造成问题的原因就出来了，如下图：

![5b139d52fdd74a7f81d6455270c34e5f~tplv-73owjymdk6-jj-mark-v1_0_0_0_0_5o6Y6YeR5oqA5pyv56S-5Yy6IEAgd3Vr_q75](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/5b139d52fdd74a7f81d6455270c34e5f~tplv-73owjymdk6-jj-mark-v1_0_0_0_0_5o6Y6YeR5oqA5pyv56S-5Yy6IEAgd3Vr_q75.webp)

**也就是说最终提交事务和释放锁的顺序有问题，按照上面的代码写法，因为当只有方法执行完了，AOP切面才会提交事务，那么如果你将上锁的代码写到被@Transactional注解的方法里面，那么提交事务永远都会处于释放锁之后，那么在释放锁之后，提交事务之前的这段时间，就会有并发问题。**

### 三、正确的写法

1. 本地锁

```java
   public void addLock(Long modelId) throws GseException {
    synchronized (this) {
        update(modelId);
    }
}

@Transactional(rollbackFor = Exception.class)
public void xxxCopy(Long modelId) throws GseException {
    synchronized (this) {
        //业务逻辑代码
    }
}
```
2. 分布式锁

```java
public void addLock(Long modelId) throws GseException {
  RLock lock = redissonClient.getLock(REDIS_COST_MODEL_ID_LOCK + modelId);
  try {
      if (lock.tryLock(20, 2, TimeUnit.SECONDS)) {
          update(modelId);
      } else {
          log.error("获取分布式锁失败");
      }
  } catch (InterruptedException e) {
      throw new RuntimeException(e);
  } finally {
      // 判断当前线程是否持有锁
      if (lock.isHeldByCurrentThread()) {
          //释放当前锁
          lock.unlock();
      }
  }
}

@Transactional(rollbackFor = Exception.class)
public void update(Long modelId) throws GseException {
    synchronized (this) {
        //业务逻辑代码
    }
}
```

这样的写法，成功避免了并发问题，**被@Transactional注解的方法，在执行完毕以后，就会提交事务，然后到了调用方法里面，再去释放锁。**

### 四、使用悲观锁和乐观锁

问题分析

1. 请求A：查询状态 → 符合条件 → 更新状态（但数据库卡顿，更新缓慢）
2. 请求B：在A完成前查询状态 → 看到的状态未被修改 → 也执行更新

这会导致：

- 数据不一致（如重复扣减库存）
- 业务逻辑错误（如订单重复支付）
- 资源超用（如优惠券被多个用户同时使用）

解决方案

1. 事务 + SELECT FOR UPDATE（推荐）

在事务中使用`SELECT FOR UPDATE`锁定行，确保更新操作的原子性：

```java

@Transactional
public void updateWithLock(Long id) {
    // 使用SELECT FOR UPDATE锁定行
    MyData data = myDataMapper.selectForUpdate(id);

    // 检查状态
    if (data.getStatus() == TARGET_STATUS) {
        // 执行更新
        data.setStatus(NEW_STATUS);
        myDataMapper.update(data);
    }
}
```

SQL映射：

```
<!-- 使用FOR UPDATE锁定行 -->
<select id="selectForUpdate" resultType="MyData">
    SELECT * FROM my_table WHERE id = #{id} FOR UPDATE
</select>
```

2. 乐观锁（使用版本号）

添加版本号字段，每次更新检查版本号：

```java
public boolean updateWithVersion(Long id) {
    MyData data = myDataMapper.findById(id);

    if (data.getStatus() != TARGET_STATUS) {
        return false;
    }

    // 带版本号的更新
    int rows = myDataMapper.updateWithVersion(
            id,
            NEW_STATUS,
            data.getVersion(),  // 当前版本
            data.getVersion() + 1 // 新版本
    );

    return rows > 0;
}
```

SQL映射：

```java
<update id="updateWithVersion">
UPDATE my_table
SET status = #{newStatus},
version = #{newVersion}
WHERE id = #{id}
AND version = #{oldVersion}
</update>
```

参考文章：[记录一次Redisson使用synchronized和分布式锁不生效的原因](https://article.juejin.cn/post/7392104848974528552)