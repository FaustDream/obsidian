# Redis三剑客（穿透、击穿、雪崩）以及解决方案

Redis的三大经典问题——缓存穿透、缓存击穿、缓存雪崩，是分布式系统中必须要解决的关键问题。让我详细解释每个问题及其解决方案。

## 1. 缓存穿透 (Cache Penetration)

**问题描述：** 大量请求查询不存在的数据，这些请求既不会命中缓存，也不会在数据库中找到结果，导致每次都要穿透到数据库层。

**典型场景：**

```java
// 恶意用户不断查询不存在的用户ID
String userId = "user_999999999";
User user = redis.get(userId);
if (user == null) {
    user = database.getUserById(userId); // 每次都会查询数据库
    // user为null，不会缓存
}
```

**解决方案：**

### 1.1 布隆过滤器 (推荐)

```java
@Component
public class UserService {
    
    @Autowired
    private BloomFilter<String> bloomFilter;
    
    public User getUserById(String userId) {
        // 先通过布隆过滤器判断
        if (!bloomFilter.mightContain(userId)) {
            return null; // 一定不存在，直接返回
        }
        
        // 可能存在，继续查询缓存和数据库
        User user = redisTemplate.opsForValue().get("user:" + userId);
        if (user == null) {
            user = userMapper.selectById(userId);
            if (user != null) {
                redisTemplate.opsForValue().set("user:" + userId, user, 30, TimeUnit.MINUTES);
            }
        }
        return user;
    }
}
```

### 1.2 缓存空值

```java
public User getUserById(String userId) {
    User user = redisTemplate.opsForValue().get("user:" + userId);
    if (user == null) {
        user = userMapper.selectById(userId);
        if (user != null) {
            redisTemplate.opsForValue().set("user:" + userId, user, 30, TimeUnit.MINUTES);
        } else {
            // 缓存空值，设置较短的过期时间
            redisTemplate.opsForValue().set("user:" + userId, "NULL", 5, TimeUnit.MINUTES);
        }
    }
    return "NULL".equals(user) ? null : user;
}
```

## 2. 缓存击穿 (Cache Breakdown)

**问题描述：** 某个热点数据的缓存过期瞬间，大量并发请求同时访问这个数据，导致请求直接打到数据库。

**典型场景：**

```java
// 热门商品信息缓存过期，大量用户同时访问
public Product getHotProduct(String productId) {
    Product product = redis.get("product:" + productId);
    if (product == null) {
        // 多个线程同时执行这里，都去查询数据库
        product = database.getProductById(productId);
        redis.set("product:" + productId, product, 3600);
    }
    return product;
}
```

**解决方案：**

### 2.1 分布式锁 (推荐)

```java
@Component
public class ProductService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public Product getProduct(String productId) {
        String cacheKey = "product:" + productId;
        String lockKey = "lock:" + productId;
        
        // 先查缓存
        Product product = (Product) redisTemplate.opsForValue().get(cacheKey);
        if (product != null) {
            return product;
        }
        
        // 获取分布式锁
        String lockValue = UUID.randomUUID().toString();
        Boolean lockAcquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, lockValue, Duration.ofSeconds(10));
            
        if (lockAcquired) {
            try {
                // 双重检查
                product = (Product) redisTemplate.opsForValue().get(cacheKey);
                if (product != null) {
                    return product;
                }
                
                // 查询数据库
                product = productMapper.selectById(productId);
                if (product != null) {
                    redisTemplate.opsForValue().set(cacheKey, product, 
                        Duration.ofMinutes(30));
                }
                return product;
                
            } finally {
                // 释放锁
                releaseLock(lockKey, lockValue);
            }
        } else {
            // 没获到锁，等待一下再重试
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return getProduct(productId); // 递归重试
        }
    }
    
    private void releaseLock(String lockKey, String lockValue) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) else return 0 end";
        redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), 
            Arrays.asList(lockKey), lockValue);
    }
}
```

### 2.2 热点数据永不过期

```java
public Product getProduct(String productId) {
    Product product = (Product) redisTemplate.opsForValue().get("product:" + productId);
    if (product != null) {
        // 检查逻辑过期时间
        if (product.getExpireTime().isAfter(LocalDateTime.now())) {
            return product;
        } else {
            // 异步更新缓存
            CompletableFuture.runAsync(() -> refreshCache(productId));
            return product; // 返回过期数据
        }
    }
    
    // 缓存不存在，同步加载
    return loadProductAndCache(productId);
}
```

## 3. 缓存雪崩 (Cache Avalanche)

**问题描述：** 大量缓存同时过期或Redis服务宕机，导致大量请求直接打到数据库，可能压垮数据库。

**解决方案：**

### 3.1 随机过期时间

```java
@Service
public class CacheService {
    
    public void setCache(String key, Object value, int baseExpireSeconds) {
        // 在基础过期时间上增加随机值，避免同时过期
        int randomExpire = baseExpireSeconds + new Random().nextInt(300); // 0-300秒随机
        redisTemplate.opsForValue().set(key, value, Duration.ofSeconds(randomExpire));
    }
    
    public void batchSetCache(Map<String, Object> dataMap, int baseExpireSeconds) {
        dataMap.forEach((key, value) -> {
            int randomExpire = baseExpireSeconds + new Random().nextInt(300);
            redisTemplate.opsForValue().set(key, value, Duration.ofSeconds(randomExpire));
        });
    }
}
```

### 3.2 多级缓存架构

```java
@Component
public class MultiLevelCacheService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    private final Cache<String, Object> localCache = Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .build();
    
    public Object get(String key) {
        // L1: 本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            return value;
        }
        
        // L2: Redis缓存
        value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            localCache.put(key, value);
            return value;
        }
        
        // L3: 数据库
        return loadFromDatabase(key);
    }
}
```

### 3.3 熔断降级

```java
@Component
public class ProductServiceWithCircuitBreaker {
    
    private final CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("productService");
    
    public Product getProduct(String productId) {
        Supplier<Product> productSupplier = CircuitBreaker
            .decorateSupplier(circuitBreaker, () -> {
                // 正常的缓存+数据库查询逻辑
                return getProductFromCacheOrDB(productId);
            });
            
        try {
            return productSupplier.get();
        } catch (CallNotPermittedException e) {
            // 熔断时返回默认值或从本地缓存获取
            return getProductFromLocalCache(productId);
        }
    }
}
```

## 综合解决方案

```java
@Service
public class ComprehensiveCacheService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private BloomFilter<String> bloomFilter;
    
    private final Cache<String, Object> localCache = Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .build();
    
    public Object getData(String key) {
        // 1. 防穿透：布隆过滤器
        if (!bloomFilter.mightContain(key)) {
            return null;
        }
        
        // 2. 多级缓存：本地缓存
        Object data = localCache.getIfPresent(key);
        if (data != null) {
            return data;
        }
        
        // 3. Redis缓存
        data = redisTemplate.opsForValue().get(key);
        if (data != null) {
            localCache.put(key, data);
            return data;
        }
        
        // 4. 防击穿：分布式锁
        String lockKey = "lock:" + key;
        String lockValue = UUID.randomUUID().toString();
        
        if (redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, Duration.ofSeconds(10))) {
            try {
                // 双重检查
                data = redisTemplate.opsForValue().get(key);
                if (data != null) {
                    return data;
                }
                
                // 数据库查询
                data = loadFromDatabase(key);
                if (data != null) {
                    // 5. 防雪崩：随机过期时间
                    int expire = 1800 + new Random().nextInt(300);
                    redisTemplate.opsForValue().set(key, data, Duration.ofSeconds(expire));
                    localCache.put(key, data);
                } else {
                    // 防穿透：缓存空值
                    redisTemplate.opsForValue().set(key, "NULL", Duration.ofMinutes(5));
                }
                
                return data;
            } finally {
                releaseLock(lockKey, lockValue);
            }
        }
        
        // 获取锁失败，短暂等待后重试
        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return getData(key);
    }
}
```

## 监控和预警

```java
@Component
public class CacheMonitor {
    
    private final MeterRegistry meterRegistry;
    
    public CacheMonitor(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    public void recordCacheHit(String cacheName) {
        Counter.builder("cache.hit")
            .tag("cache", cacheName)
            .register(meterRegistry)
            .increment();
    }
    
    public void recordCacheMiss(String cacheName) {
        Counter.builder("cache.miss")
            .tag("cache", cacheName)
            .register(meterRegistry)
            .increment();
    }
    
    public void recordDatabaseQuery(String operation, long duration) {
        Timer.builder("database.query")
            .tag("operation", operation)
            .register(meterRegistry)
            .record(duration, TimeUnit.MILLISECONDS);
    }
}
```

这三个问题在高并发系统中经常同时出现，需要综合运用多种策略来解决。关键是要根据业务场景选择合适的方案，并建立完善的监控体系来及时发现和处理问题。