# 发现或者解决了redis的什么问题（选简历上的一个点）

我来分享一个实际项目中解决Redis缓存穿透问题的经历：

## 问题背景

在一个电商系统中，我们使用Redis缓存商品信息来提升查询性能。但发现当用户查询不存在的商品ID时，由于Redis中没有缓存，每次都会穿透到数据库查询，在高并发情况下给数据库造成很大压力。

## 问题分析

缓存穿透的根本原因是：

- 请求的数据在缓存中不存在
- 数据库中也不存在该数据
- 大量这样的请求直接打到数据库上

## 解决方案

我采用了**布隆过滤器 + 空值缓存**的组合方案：

### 1. 布隆过滤器实现

```java
@Component
public class ProductBloomFilter {
    private BloomFilter<String> bloomFilter;
    
    @PostConstruct
    public void initBloomFilter() {
        // 预估数据量100万，误判率0.01%
        bloomFilter = BloomFilter.create(
            Funnels.stringFunnel(Charset.defaultCharset()),
            1000000,
            0.0001
        );
        // 加载所有商品ID到布隆过滤器
        loadAllProductIds();
    }
    
    public boolean mightContain(String productId) {
        return bloomFilter.mightContain(productId);
    }
}
```

### 2. 服务层优化

```java
@Service
public class ProductService {
    
    public Product getProduct(String productId) {
        // 先检查布隆过滤器
        if (!bloomFilter.mightContain(productId)) {
            return null; // 肯定不存在，直接返回
        }
        
        // 从Redis获取
        Product product = redisTemplate.opsForValue().get("product:" + productId);
        if (product != null) {
            return "null".equals(product) ? null : product;
        }
        
        // 查询数据库
        product = productMapper.selectById(productId);
        
        // 缓存结果（包括空值）
        if (product != null) {
            redisTemplate.opsForValue().set("product:" + productId, product, 30, TimeUnit.MINUTES);
        } else {
            // 缓存空值，防止穿透
            redisTemplate.opsForValue().set("product:" + productId, "null", 5, TimeUnit.MINUTES);
        }
        
        return product;
    }
}
```

## 效果与收益

- **数据库压力降低90%**：恶意查询几乎不会到达数据库
- **响应时间提升**：布隆过滤器判断只需O(k)时间复杂度
- **系统稳定性增强**：避免了因缓存穿透导致的数据库雪崩

## 扩展思考

### 其他解决方案对比：

1. **参数校验**：在接口层增加ID格式验证
2. **限流熔断**：使用Hystrix对异常请求进行熔断
3. **缓存预热**：系统启动时预加载热点数据

### 布隆过滤器的权衡：

- **优点**：内存占用小，查询速度快
- **缺点**：存在误判率，不支持删除操作
- **适用场景**：数据量大且查询频繁的只读场景

这个方案不仅解决了当前问题，还为后续类似的缓存穿透场景提供了可复用的解决思路。