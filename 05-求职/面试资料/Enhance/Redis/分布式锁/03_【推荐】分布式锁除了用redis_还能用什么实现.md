# 分布式锁除了用redis，还能用什么实现

分布式锁除了Redis，还有多种实现方式，每种都有自己的特点和适用场景：

## 1. 基于数据库实现

**乐观锁方式：**

```sql
-- 使用版本号实现乐观锁
UPDATE resource SET version = version + 1, data = 'new_data' 
WHERE id = 1 AND version = 100;
```

**悲观锁方式：**

```sql
-- 使用SELECT FOR UPDATE实现悲观锁
BEGIN;
SELECT * FROM lock_table WHERE resource_id = 'order_123' FOR UPDATE;
-- 执行业务逻辑
COMMIT;
```

**优点：** 实现简单，事务性强，数据一致性好 **缺点：** 性能相对较低，存在单点故障风险

## 2. 基于Zookeeper实现

Zookeeper通过临时有序节点实现分布式锁：

```java
public class ZookeeperLock {
    public boolean tryLock(String lockPath) {
        try {
            // 创建临时有序节点
            String currentPath = zk.create(lockPath + "/", 
                new byte[0], OPEN_ACL_UNSAFE, EPHEMERAL_SEQUENTIAL);
            
            // 获取所有子节点并排序
            List<String> children = zk.getChildren(lockPath, false);
            Collections.sort(children);
            
            // 如果当前节点是最小的，则获得锁
            if (currentPath.endsWith(children.get(0))) {
                return true;
            }
            
            // 否则监听前一个节点
            String prevNode = null;
            for (int i = 0; i < children.size(); i++) {
                if (currentPath.endsWith(children.get(i))) {
                    prevNode = children.get(i - 1);
                    break;
                }
            }
            
            // 监听前一个节点的删除事件
            zk.exists(lockPath + "/" + prevNode, watcher);
            return false;
        } catch (Exception e) {
            return false;
        }
    }
}
```

**优点：** 强一致性，自动故障恢复，避免惊群效应 **缺点：** 复杂度较高，对Zookeeper集群依赖性强

## 3. 基于Etcd实现

Etcd通过租约（Lease）和事务实现分布式锁：

```go
// 使用Etcd实现分布式锁的核心逻辑
func (m *EtcdMutex) Lock() error {
    // 创建租约
    lease, err := m.client.Grant(context.TODO(), int64(m.ttl))
    if err != nil {
        return err
    }
    
    // 尝试获取锁
    key := fmt.Sprintf("%s/%s", m.prefix, m.key)
    txn := m.client.Txn(context.TODO())
    txn.If(clientv3.Compare(clientv3.CreateRevision(key), "=", 0))
    txn.Then(clientv3.OpPut(key, m.value, clientv3.WithLease(lease.ID)))
    
    resp, err := txn.Commit()
    if err != nil {
        return err
    }
    
    if !resp.Succeeded {
        return errors.New("lock already exists")
    }
    
    return nil
}
```

**优点：** 高可用，强一致性，支持TTL自动过期 **缺点：** 运维复杂度较高

## 4. 基于Consul实现

Consul通过Session和KV存储实现分布式锁：

```java
public class ConsulLock {
    public boolean tryLock(String key, String sessionId) {
        try {
            // 尝试获取锁
            PutParams params = new PutParams();
            params.setAcquireSession(sessionId);
            
            Response<Boolean> response = consulClient.setKVValue(key, 
                "locked", params);
            
            return response.getValue();
        } catch (Exception e) {
            return false;
        }
    }
}
```

**优点：** 内置健康检查，服务发现集成，自动故障恢复 **缺点：** 相对复杂，需要理解Session机制

## 5. 基于文件系统实现

通过文件系统的原子操作实现简单的分布式锁：

```java
public class FileLock {
    private File lockFile;
    private FileChannel channel;
    private FileLock lock;
    
    public boolean tryLock() {
        try {
            channel = new RandomAccessFile(lockFile, "rw").getChannel();
            lock = channel.tryLock();
            return lock != null;
        } catch (IOException e) {
            return false;
        }
    }
    
    public void unlock() {
        try {
            if (lock != null) {
                lock.release();
            }
            if (channel != null) {
                channel.close();
            }
        } catch (IOException e) {
            // 处理异常
        }
    }
}
```

**优点：** 实现简单，无需额外中间件 **缺点：** 只适用于单机或共享文件系统环境

## 各种实现方式对比

|实现方式|性能|一致性|可用性|复杂度|适用场景|
|---|---|---|---|---|---|
|Redis|高|最终一致性|高|低|高并发，容错性要求不高|
|Database|中|强一致性|中|低|数据一致性要求高|
|Zookeeper|中|强一致性|高|高|对一致性要求极高|
|Etcd|中|强一致性|高|高|微服务环境|
|Consul|中|强一致性|高|中|需要服务发现的场景|
|文件系统|高|强一致性|低|低|单机或共享存储环境|

## 选择建议

1. **高并发场景**：优先选择Redis，配合Redisson等客户端
2. **强一致性要求**：选择Zookeeper或Etcd
3. **简单业务场景**：数据库实现足够
4. **微服务架构**：Consul或Etcd更合适
5. **资源受限环境**：文件系统实现

每种方案都要考虑CAP理论的权衡，根据业务需求选择最适合的实现方式。