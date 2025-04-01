---
title: 使用Redis和RedisTemplate设计当日访问量和访问人数统计
date: 2025-04-01 22:32:07
tags:
---

> 在Java中使用Redis和RedisTemplate来统计当日访问量和访问人数（UV）是一个常见的需求。下面我将提供一个完整的实现方案。

## 1. 设计思路

• **访问量(PV)**：使用Redis的INCR命令统计总访问次数
• **访问人数(UV)**：使用Redis的HyperLogLog数据结构统计独立访客数
• **过期时间**：设置当日24点自动过期

## 2. 实现代码

```java
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.util.concurrent.TimeUnit;

@Service
public class VisitStatisticsService {

    private final RedisTemplate<String, String> redisTemplate;

    public VisitStatisticsService(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    // 获取当日PV的key
    private String getTodayPVKey() {
        return "stat:pv:" + LocalDate.now().toString();
    }

    // 获取当日UV的key
    private String getTodayUVKey() {
        return "stat:uv:" + LocalDate.now().toString();
    }

    // 获取当天剩余的秒数（到24点的秒数）
    private long getRemainingSecondsToday() {
        ZonedDateTime now = ZonedDateTime.now(ZoneId.systemDefault());
        ZonedDateTime midnight = now.toLocalDate().plusDays(1).atStartOfDay(ZoneId.systemDefault());
        return midnight.toEpochSecond() - now.toEpochSecond();
    }

    /**
     * 记录访问
     * @param userId 用户ID（可以为IP地址或其他唯一标识）
     */
    public void recordVisit(String userId) {
        // 获取当天剩余的秒数
        long remainingSeconds = getRemainingSecondsToday();
        
        // 记录PV（访问量）
        ValueOperations<String, String> valueOps = redisTemplate.opsForValue();
        String pvKey = getTodayPVKey();
        valueOps.increment(pvKey);
        redisTemplate.expire(pvKey, remainingSeconds, TimeUnit.SECONDS);
        
        // 记录UV（独立访客）
        String uvKey = getTodayUVKey();
        redisTemplate.opsForHyperLogLog().add(uvKey, userId);
        redisTemplate.expire(uvKey, remainingSeconds, TimeUnit.SECONDS);
    }

    /**
     * 获取当日访问量(PV)
     */
    public Long getTodayPV() {
        String pvKey = getTodayPVKey();
        String value = redisTemplate.opsForValue().get(pvKey);
        return value == null ? 0L : Long.parseLong(value);
    }

    /**
     * 获取当日访问人数(UV)
     */
    public Long getTodayUV() {
        String uvKey = getTodayUVKey();
        return redisTemplate.opsForHyperLogLog().size(uvKey);
    }

    /**
     * 获取历史某天的访问量(PV)
     * @param date 日期
     */
    public Long getPVByDate(LocalDate date) {
        String pvKey = "stat:pv:" + date.toString();
        String value = redisTemplate.opsForValue().get(pvKey);
        return value == null ? 0L : Long.parseLong(value);
    }

    /**
     * 获取历史某天的访问人数(UV)
     * @param date 日期
     */
    public Long getUVByDate(LocalDate date) {
        String uvKey = "stat:uv:" + date.toString();
        return redisTemplate.opsForHyperLogLog().size(uvKey);
    }
}
```

## 3. 使用示例

```java
@RestController
@RequestMapping("/api/visit")
public class VisitController {

    private final VisitStatisticsService visitStatisticsService;

    public VisitController(VisitStatisticsService visitStatisticsService) {
        this.visitStatisticsService = visitStatisticsService;
    }

    @GetMapping("/record")
    public String recordVisit(@RequestParam String userId) {
        visitStatisticsService.recordVisit(userId);
        return "Visit recorded for user: " + userId;
    }

    @GetMapping("/stats")
    public Map<String, Object> getStats() {
        Map<String, Object> result = new HashMap<>();
        result.put("pv", visitStatisticsService.getTodayPV());
        result.put("uv", visitStatisticsService.getTodayUV());
        return result;
    }
}
```

## 4. 设计说明

1. **PV统计**：
   • 使用简单的计数器（INCR命令）
   • 每个访问请求调用一次INCR
   • 键格式：`stat:pv:yyyy-MM-dd`

2. **UV统计**：
   • 使用HyperLogLog数据结构
   • 内存占用小（每个HyperLogLog约12KB）
   • 误差率约0.81%
   • 键格式：`stat:uv:yyyy-MM-dd`

3. **过期时间**：
   • 自动计算到当天24点的剩余秒数
   • 设置键的过期时间
   • 避免数据无限增长

4. **扩展性**：
   • 可以轻松扩展为按小时、周、月统计
   • 可以添加用户行为分析（如访问页面、停留时间等）

## 5. 性能优化

1. **批量操作**：如果有批量记录需求，可以使用pipeline提高性能
2. **持久化**：如果需要长期保存数据，可以定期将Redis数据持久化到数据库
3. **分布式**：此方案天然支持分布式环境

## 6. 注意事项

1. HyperLogLog有约0.81%的误差率，如果要求精确计数，可以使用Set数据结构（但内存消耗更大）
2. Redis重启可能导致数据丢失，重要数据应考虑持久化方案
3. 在高并发场景下，INCR命令是原子操作，无需担心并发问题

这个实现方案简单高效，适合大多数Web应用的访问统计需求。
