# Redis常见的数据类型？底层结构？

Redis有5种基本数据类型和几种特殊数据类型。让我详细介绍它们的特点和底层实现：

## 五种基本数据类型

### 1. String（字符串）

**特点：** 最基础的数据类型，可以存储字符串、数字或二进制数据 **底层结构：** SDS（Simple Dynamic String）

- 动态字符串，相比C字符串有长度信息，避免缓冲区溢出
- 支持预分配空间，减少内存重新分配次数

**应用场景：** 缓存、计数器、分布式锁

### 2. Hash（哈希）

**特点：** 键值对集合，类似Java的HashMap **底层结构：**

- 数据量小时：ziplist（压缩列表）
- 数据量大时：hashtable（哈希表）

**应用场景：** 存储对象信息，如用户信息

### 3. List（列表）

**特点：** 有序的字符串列表，支持头尾操作 **底层结构：**

- Redis 3.2前：ziplist + linkedlist
- Redis 3.2后：quicklist（ziplist和linkedlist的混合）

**应用场景：** 消息队列、最新动态列表

### 4. Set（集合）

**特点：** ==无序==且唯一的字符串集合 **底层结构：**

- 数据量小且都是整数：intset（整数集合）
- 其他情况：hashtable

**应用场景：** 标签系统、共同好友

### 5. ZSet（有序集合）

**特点：** ==有序==且唯一，每个元素关联一个分数 **底层结构：**

- 数据量小：ziplist
- 数据量大：skiplist（跳跃表）+ hashtable

**应用场景：** 排行榜、延时队列

## 特殊数据类型

### 1. Bitmap（位图）

基于String实现，用于处理二进制位操作 **应用：** 用户签到、在线状态统计

### 2. HyperLogLog

用于基数统计的概率数据结构 **应用：** UV统计、去重计数

### 3. GEO

基于==ZSet==实现的地理位置数据类型 **应用：** 附近的人、地理围栏

### 4. Stream

消息流数据类型 **应用：** 消息队列、事件溯源

## 底层优化机制

### 编码转换

Redis会根据数据量自动选择最优的底层结构：

- **ziplist：** 内存紧凑，但操作复杂度高
- **hashtable/skiplist：** 操作效率高，但内存占用大

### 关键阈值

```
hash-max-ziplist-entries 512    # Hash使用ziplist的最大元素数
hash-max-ziplist-value 64       # Hash使用ziplist的最大值长度
list-max-ziplist-size -2        # List每个节点的最大内存
zset-max-ziplist-entries 128    # ZSet使用ziplist的最大元素数
```

## 扩展知识

### 内存优化策略

1. **压缩列表（ziplist）：** 连续内存存储，节省指针开销
2. **整数集合（intset）：** 专门存储整数的紧凑结构
3. **跳跃表（skiplist）：** 多层链表，查找效率O(logN)

### 选型建议

- 简单缓存：String
- 对象存储：Hash
- 队列场景：List
- 去重场景：Set
- 排序场景：ZSet

这种设计让Redis在保证性能的同时，最大化内存利用率，这也是Redis成为高性能缓存首选的重要原因。