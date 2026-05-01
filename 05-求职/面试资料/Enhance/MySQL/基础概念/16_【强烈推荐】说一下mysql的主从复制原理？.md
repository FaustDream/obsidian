MySQL主从复制的核心是两个日志文件binlog和relaylog，记录了所有的DDL语句和DML语句。

大概过程是：
1. 主库在事务提交时，会把数据变更记录在二进制日志文件 Binlog 中。
2. 从库读取主库的日志文件Binlog，写入到自己的中继日志 RelayLog 中。
3. 从库重做 relaylog 文件中的操作，保证主库和从库的数据同步。

-----
-----

MySQL主从复制是一个非常重要的数据库技术，让我用最通俗的方式为你解释一下它的原理。

## 核心概念

MySQL主从复制就像是"老师教学生"的过程：

- **主库（Master）**：相当于老师，负责处理所有的写操作
- **从库（Slave）**：相当于学生，从主库那里学习并复制数据

## 复制原理的三个关键步骤

### 1. 主库记录操作日志（Binary Log）

```sql
-- 当主库执行这样的操作时：
INSERT INTO users (name, age) VALUES ('张三', 25);
UPDATE products SET price = 100 WHERE id = 1;
```

主库会把这些操作记录到 **二进制日志（binlog）** 中，就像老师把讲课内容写在黑板上。

### 2. 从库读取并记录（Relay Log）

从库有两个关键线程：

- **IO线程**：像学生一样，主动向主库请求获取binlog内容
- **SQL线程**：负责执行从主库复制过来的SQL语句

从库会把从主库获取的binlog内容先保存到 **中继日志（relay log）** 中。

### 3. 从库重放操作

从库的SQL线程读取relay log中的内容，在从库上重新执行一遍，从而保持与主库的数据一致。

## 复制过程图示

```
主库 (Master)
    ↓ (写入binlog)
[Binary Log] → IO线程 → 从库 [Relay Log] → SQL线程 → 从库数据
```

## 三种复制模式

### 1. 异步复制（默认）

- 主库执行完就返回，不等从库
- 性能最好，但可能有数据丢失风险

### 2. 半同步复制

- 主库等待至少一个从库确认收到binlog后才返回
- 平衡了性能和数据安全

### 3. 同步复制

- 主库等待所有从库都执行完才返回
- 最安全但性能较差

## 实际应用场景

### 读写分离

```java
// 写操作走主库
@Master
public void createUser(User user) {
    userDao.insert(user);
}

// 读操作走从库
@Slave
public List<User> findUsers() {
    return userDao.selectAll();
}
```

### 数据备份

从库可以作为主库的实时备份，当主库出现故障时可以快速切换。

## 常见面试延伸问题

### 1. 主从延迟问题

**原因**：网络延迟、从库性能不足、大事务等 

**解决方案**：
- 使用SSD提升从库IO性能
- 优化网络环境
- 避免大事务，拆分为小事务
- 使用并行复制

### 2. 数据一致性问题

```java
// 可能出现的问题
userService.createUser(user);  // 写入主库
User result = userService.getUser(id);  // 从从库读，可能读不到

// 解决方案：强制读主库
@Master
public User getUserAfterCreate(Long id) {
    return userDao.selectById(id);
}
```

### 3. 主从切换（故障转移）

当主库宕机时：

1. 停止应用写入
2. 确保从库数据完整
3. 将从库提升为主库
4. 修改应用配置指向新主库

这个原理在实际工作中非常重要，特别是在高并发、高可用的系统设计中。你还想了解哪个方面的细节吗？