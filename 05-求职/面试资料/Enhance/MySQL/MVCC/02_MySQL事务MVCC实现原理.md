# MySQL事务MVCC实现原理

MVCC（Multi-Version Concurrency Control，多版本并发控制）是MySQL中InnoDB存储引擎解决并发读写问题的核心机制。让我详细解释其实现原理：

## 1. MVCC的基本概念

MVCC通过保存数据的多个版本来实现并发控制，让读操作不阻塞写操作，写操作也不阻塞读操作，从而提高数据库的并发性能。

## 2. 核心实现机制

### 2.1 隐藏字段

InnoDB为每行记录添加了三个隐藏字段：

```sql
-- 每行记录的隐藏字段结构
DB_TRX_ID    -- 事务ID，记录最后一次修改该行的事务ID
DB_ROLL_PTR  -- 回滚指针，指向undo log中的历史版本
DB_ROW_ID    -- 行ID，如果表没有主键时使用
```

### 2.2 Undo Log（回滚日志）

Undo Log存储了数据的历史版本，形成版本链：

```
当前版本 -> 历史版本1 -> 历史版本2 -> ... -> 原始版本
```

每个版本通过DB_ROLL_PTR指针连接，形成一个版本链表。

### 2.3 ReadView（一致性视图）

ReadView是事务在某个时刻看到的数据库状态快照，包含：

```java
class ReadView {
    long m_low_limit_id;     // 当前活跃事务ID的最小值
    long m_up_limit_id;      // 下一个要分配的事务ID
    List<Long> m_ids;        // 当前活跃的事务ID列表
    long m_creator_trx_id;   // 创建该ReadView的事务ID
}
```

## 3. 版本可见性判断规则

对于版本链中的每个版本，通过以下规则判断是否可见：

```java
public boolean isVisible(long trxId, ReadView readView) {
    // 1. 如果记录的事务ID等于当前事务ID，可见
    if (trxId == readView.m_creator_trx_id) {
        return true;
    }
    
    // 2. 如果记录的事务ID小于最小活跃事务ID，说明事务已提交，可见
    if (trxId < readView.m_low_limit_id) {
        return true;
    }
    
    // 3. 如果记录的事务ID大于等于下一个事务ID，说明事务是后启动的，不可见
    if (trxId >= readView.m_up_limit_id) {
        return false;
    }
    
    // 4. 如果记录的事务ID在活跃事务列表中，不可见
    if (readView.m_ids.contains(trxId)) {
        return false;
    }
    
    // 5. 其他情况可见
    return true;
}
```

## 4. 不同隔离级别下的ReadView生成时机

### 4.1 READ COMMITTED（读已提交）

每次读取都生成新的ReadView，能看到最新提交的数据。

```java
// RC级别：每次查询都创建新的ReadView
public Result select() {
    ReadView readView = createReadView(); // 每次都创建
    return queryWithReadView(readView);
}
```

### 4.2 REPEATABLE READ（可重复读）

事务开始时生成ReadView，整个事务过程中复用，保证可重复读。

```java
// RR级别：事务开始时创建ReadView，后续复用
public class Transaction {
    private ReadView readView;
    
    public Result select() {
        if (readView == null) {
            readView = createReadView(); // 首次创建
        }
        return queryWithReadView(readView); // 复用同一个ReadView
    }
}
```

## 5. 具体查询流程

```java
public Record queryRecord(String key, ReadView readView) {
    // 1. 从聚簇索引获取最新记录
    Record record = getLatestRecord(key);
    
    // 2. 沿着版本链查找可见版本
    while (record != null) {
        if (isVisible(record.trxId, readView)) {
            return record; // 找到可见版本
        }
        // 3. 通过undo log获取历史版本
        record = getHistoryVersion(record.rollPtr);
    }
    
    return null; // 没有可见版本
}
```

## 6. 实际案例演示

假设有以下事务执行序列：

```sql
-- 初始数据：id=1, name='张三', trx_id=100
-- 事务A（ID=200）和事务B（ID=300）并发执行

-- 事务A：
START TRANSACTION;
UPDATE user SET name='李四' WHERE id=1; -- 修改但未提交

-- 事务B：
START TRANSACTION;
SELECT * FROM user WHERE id=1; -- 应该看到什么？
```

版本链结构：

```
当前版本：name='李四', trx_id=200, roll_ptr->历史版本
历史版本：name='张三', trx_id=100, roll_ptr->NULL
```

事务B的ReadView：

```java
ReadView {
    m_low_limit_id = 200,    // 最小活跃事务ID
    m_up_limit_id = 301,     // 下一个事务ID
    m_ids = [200, 300],      // 活跃事务列表
    m_creator_trx_id = 300   // 当前事务ID
}
```

可见性判断：

- 当前版本（trx_id=200）在活跃事务列表中，不可见
- 历史版本（trx_id=100）< 200，可见
- 因此事务B看到name='张三'

## 7. MVCC的优势

1. **读写不阻塞**：读操作不需要加锁，写操作不会阻塞读操作
2. **高并发性能**：减少锁竞争，提高系统吞吐量
3. **一致性保证**：通过版本控制保证事务的一致性读取

## 8. 相关知识扩展

### 8.1 与锁机制的关系

MVCC主要解决读-写冲突，但写-写冲突仍需要锁机制（如行锁、间隙锁）来解决。

### 8.2 Undo Log的清理

MySQL通过purge线程定期清理不再需要的undo log版本，避免版本链过长影响性能。

### 8.3 快照读与当前读

- 快照读：普通SELECT使用MVCC
- 当前读：SELECT...FOR UPDATE、UPDATE、DELETE使用锁机制

这就是MySQL InnoDB存储引擎中MVCC的完整实现原理，它是保证数据库高并发性能的关键技术之一。