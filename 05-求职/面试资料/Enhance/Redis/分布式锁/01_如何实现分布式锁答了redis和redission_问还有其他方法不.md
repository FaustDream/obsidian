# 如何实现分布式锁？答了redis和redission，问还有其他方法不？

除了Redis和Redisson，还有多种实现分布式锁的方法：

## 1. 基于数据库的分布式锁

**唯一索引实现**：

```sql
CREATE TABLE distributed_lock (
    lock_name VARCHAR(64) PRIMARY KEY,
    holder VARCHAR(64),
    expire_time TIMESTAMP
);

-- 获取锁
INSERT INTO distributed_lock (lock_name, holder, expire_time) 
VALUES ('order_lock', 'server1', NOW() + INTERVAL 30 SECOND);

-- 释放锁
DELETE FROM distributed_lock WHERE lock_name = 'order_lock' AND holder = 'server1';
```

**优点**：实现简单，利用数据库ACID特性 **缺点**：性能较差，可能成为瓶颈

## 2. 基于ZooKeeper的分布式锁

```java
public class ZookeeperLock {
    private CuratorFramework client;
    private InterProcessMutex lock;
    
    public boolean tryLock(String lockPath, long timeout) {
        lock = new InterProcessMutex(client, lockPath);
        try {
            return lock.acquire(timeout, TimeUnit.SECONDS);
        } catch (Exception e) {
            return false;
        }
    }
    
    public void unlock() {
        try {
            lock.release();
        } catch (Exception e) {
            // 处理异常
        }
    }
}
```

**优点**：强一致性，有序节点天然支持公平锁 **缺点**：性能不如Redis，复杂度较高

## 3. 基于etcd的分布式锁

```java
public class EtcdLock {
    private EtcdClient client;
    
    public boolean tryLock(String key, String value, long leaseTime) {
        try {
            // 创建租约
            long leaseId = client.getLeaseClient().grant(leaseTime).get().getID();
            
            // 尝试获取锁
            TxnResponse response = client.getKVClient().txn()
                .If(new Cmp(ByteSequence.from(key.getBytes()), 
                    Cmp.Op.EQUAL, CmpTarget.createRevision(0)))
                .Then(Op.put(ByteSequence.from(key.getBytes()), 
                    ByteSequence.from(value.getBytes()), 
                    PutOption.newBuilder().withLeaseId(leaseId).build()))
                .commit().get();
                
            return response.isSucceeded();
        } catch (Exception e) {
            return false;
        }
    }
}
```

## 4. 基于文件系统的分布式锁

```java
public class FileLock {
    private String lockFile;
    private FileChannel channel;
    private FileLock lock;
    
    public boolean tryLock(String fileName) {
        try {
            File file = new File(fileName);
            channel = new RandomAccessFile(file, "rw").getChannel();
            lock = channel.tryLock();
            return lock != null;
        } catch (Exception e) {
            return false;
        }
    }
}
```

## 5. 基于Consul的分布式锁

```java
public class ConsulLock {
    private ConsulClient consul;
    
    public boolean tryLock(String key, String sessionId) {
        try {
            PutParams params = new PutParams();
            params.setAcquireSession(sessionId);
            
            Response<Boolean> response = consul.setKVValue(key, "locked", params);
            return response.getValue();
        } catch (Exception e) {
            return false;
        }
    }
}
```

## 各种方案对比

|方案|性能|一致性|复杂度|适用场景|
|---|---|---|---|---|
|Redis|高|最终一致|低|高并发，对一致性要求不严格|
|ZooKeeper|中|强一致|高|对一致性要求高，并发量中等|
|数据库|低|强一致|低|并发量小，已有数据库基础设施|
|etcd|中|强一致|中|云原生环境，需要强一致性|
|文件系统|低|强一致|低|单机多进程场景|

## 实际选择建议

1. **高并发场景**：优选Redis + Redisson
2. **强一致性要求**：选择ZooKeeper或etcd
3. **简单场景**：可考虑数据库实现
4. **云原生环境**：etcd是不错选择
5. **已有基础设施**：基于现有中间件选择

每种方案都有其适用场景，需要根据具体的业务需求、性能要求和技术栈来选择最合适的实现方式。