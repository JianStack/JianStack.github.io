---
title: Redis缓存架构设计与实战解析(嗨评星球)
date: 2025-07-14
updated: 2025-07-14
categories: 技术分享
tags:
  - reids
  - 高并发
  - 数据库
  - 开发技巧
top: 1
---

# 高并发场景下的Redis缓存架构设计与实战解析（嗨评星球）

## 一、缓存穿透、击穿与雪崩：问题与解决方案

### 1. 缓存穿透（Cache Penetration）

**问题**：大量请求查询数据库中不存在的数据，导致请求直接穿透到数据库

**解决方案**： 

```
// 在CacheClient中的实现
if (r == null) {
    // 缓存空值并设置较短过期时间
    stringRedisTemplate.opsForValue().set(key,"",CACHE_NULL_TTL, TimeUnit.MINUTES);
    return null;
}
```

### 2. 缓存击穿（Cache Breakdown）

**问题**：热点key过期瞬间，大量请求直接打到数据库

**解决方案**：

- **互斥锁方案**：

```
// 使用Redis分布式锁控制重建过程
boolean isLock = tryLock(lockKey);
if (!isLock) {
    Thread.sleep(50);
    return queryWithMutex(id);
}
```

- **逻辑过期方案**：

```
// 使用逻辑过期时间+异步重建
if (expireTime.isAfter(LocalDateTime.now())) {
    return shop; // 未过期直接返回
}
// 过期则异步重建
CACHE_REBUILD_EXECUTOR.submit(() -> {
    // 重建缓存
    this.saveShop2Redis(id,20L);
});
```

### 3. 缓存雪崩（Cache Avalanche）

**解决方案**：

- 随机设置缓存过期时间
- 集群部署保证高可用
- 预热重要数据

## 二、Redis高级应用技巧

### 1. 逻辑过期设计

```
public class RedisData {
    private LocalDateTime expireTime; // 逻辑过期时间
    private Object data;             // 业务数据
}

// 设置逻辑过期缓存
public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit timeUnit){
    RedisData redisData = new RedisData();
    redisData.setData(value);
    redisData.setExpireTime(LocalDateTime.now().plusSeconds(timeUnit.toSeconds(time)));
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
}
```

### 2. 分布式锁实现

```
private boolean tryLock(String key){
    // 使用SETNX命令实现分布式锁
    return BooleanUtil.isTrue(
        stringRedisTemplate.opsForValue()
            .setIfAbsent(key, "1", 10, TimeUnit.SECONDS)
    );
}

private void unLock(String key){
    stringRedisTemplate.delete(key);
}
```

### 3. 双写一致性策略

```
@Override
@Transactional
public Result update(Shop shop) {
    // 先更新数据库
    updateById(shop);
    // 再删除缓存
    stringRedisTemplate.delete(CACHE_SHOP_KEY + shop.getId());
    return Result.ok();
}
```

## 三、通用缓存模板设计

### 1. 函数式编程解耦

```
public <R,ID> R queryWithPassThrough(
    String keyPrefix, 
    ID id, 
    Class<R> type, 
    Function<ID,R> dbFallback, // 核心：数据库查询逻辑回调
    Long time, 
    TimeUnit unit) {
    
    // ...缓存逻辑
    R r = dbFallback.apply(id); // 执行数据库查询
    // ...后续处理
}
```

### 2. 泛型应用

```
public <R,ID> R queryWithLogicalExpire(
    String keyPrefix,
    ID id,
    Class<R> type, // 泛型类类型
    Function<ID,R> dbFallback,
    Long time, 
    TimeUnit unit) {
    
    // 泛型反序列化
    R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
    // ...
}
```

## 四、高并发优化实践

### 1. 线程池异步重建

```
private static final ExecutorService CACHE_REBUILD_EXECUTOR = 
    Executors.newFixedThreadPool(10);

CACHE_REBUILD_EXECUTOR.submit(() -> {
    try {
        // 异步重建缓存
        R r1 = dbFallback.apply(id);
        this.setWithLogicalExpire(key, r1, time, unit);
    } finally {
        unLock(lockKey);
    }
});
```

### 2. 双重检查锁（Double-Check）

```
// 在获取锁后再次检查缓存
String latestShopJson = stringRedisTemplate.opsForValue().get(key);
if (StrUtil.isNotBlank(latestShopJson)) {
    // 如果已被其他线程重建，直接返回
    return JSONUtil.toBean(/*...*/);
}
```

## 五、Redis使用最佳实践

1. **键名设计**：使用业务前缀（如`CACHE_SHOP_KEY`）避免冲突
2. **序列化**：使用JSON格式存储复杂对象
3. **过期策略**：
   - 正常数据：`CACHE_SHOP_TTL`（30分钟）
   - 空值数据：`CACHE_NULL_TTL`（2分钟）
4. **连接池**：合理配置连接池参数
5. **监控**：监控缓存命中率、内存使用等关键指标

## 六、性能对比

| 方案         | 优点                   | 缺点               | 适用场景             |
| :----------- | :--------------------- | :----------------- | :------------------- |
| 缓存穿透方案 | 简单有效，防止恶意攻击 | 可能缓存大量空值   | 查询不存在数据的场景 |
| 互斥锁方案   | 保证强一致性           | 线程阻塞，性能中等 | 对一致性要求高的场景 |
| 逻辑过期方案 | 高并发性能好，非阻塞   | 短暂数据不一致     | 高并发读场景         |

## 七、总结与展望

本文介绍的高并发缓存架构实现了：

1. 多级缓存策略应对不同场景
2. 泛型+函数式编程实现缓存模板复用
3. 异步线程池提升系统吞吐量
4. 分布式锁保证数据一致性

**未来优化方向**：

1. 引入本地缓存（Caffeine）作为L1缓存
2. 实现缓存预热机制
3. 添加熔断降级策略
4. 结合布隆过滤器优化缓存穿透防护

通过合理的Redis缓存设计，系统QPS可从数据库直接支撑的2000提升到Redis支撑的10万+，同时保证系统的高可用性和数据一致性。