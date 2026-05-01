# Innodb事务隔离级别，默认事务

InnoDB的事务隔离级别是数据库面试中的经典问题，让我为你详细解答：

## InnoDB的四种事务隔离级别

### 1. READ UNCOMMITTED（读未提交）

- **特点**：可以读取其他事务未提交的数据
- **问题**：会产生脏读、不可重复读、幻读
- **实际应用**：几乎不使用，因为数据一致性很差

### 2. READ COMMITTED（读已提交）

- **特点**：只能读取已经提交的数据
- **问题**：避免了脏读，但仍存在不可重复读和幻读
- **实际应用**：Oracle、SQL Server的默认级别

### 3. REPEATABLE READ（可重复读）

- **特点**：在同一事务中多次读取同一数据结果一致
- **问题**：避免了脏读和不可重复读，但理论上仍可能出现幻读
- **重要**：这是MySQL InnoDB的**默认隔离级别**

### 4. SERIALIZABLE（串行化）

- **特点**：最高隔离级别，完全串行化执行
- **问题**：避免所有并发问题，但性能最差
- **实际应用**：很少使用，除非对一致性要求极高

## InnoDB的默认事务隔离级别

**答案：REPEATABLE READ（可重复读）**

可以通过以下SQL查看：

```sql
-- 查看全局隔离级别
SELECT @@global.transaction_isolation;

-- 查看当前会话隔离级别
SELECT @@session.transaction_isolation;
```

## 关键知识点扩展

### 1. InnoDB如何实现REPEATABLE READ

- **MVCC（多版本并发控制）**：通过undo log维护数据的多个版本
- **ReadView机制**：为每个事务创建一致性视图
- **间隙锁（Gap Lock）**：防止幻读的发生

### 2. 三种读取方式

```sql
-- 当前读（会加锁）
SELECT * FROM table WHERE id = 1 FOR UPDATE;
SELECT * FROM table WHERE id = 1 LOCK IN SHARE MODE;

-- 快照读（不加锁，基于MVCC）
SELECT * FROM table WHERE id = 1;
```

### 3. 锁机制配合

- **记录锁（Record Lock）**：锁定具体记录
- **间隙锁（Gap Lock）**：锁定记录之间的间隙
- **Next-Key Lock**：记录锁+间隙锁的组合

### 4. 实际应用场景

- **金融系统**：通常使用REPEATABLE READ保证数据一致性
- **高并发系统**：可能降级到READ COMMITTED提高性能
- **报表系统**：可能使用SERIALIZABLE确保数据完整性

### 5. 常见面试追问

**Q：为什么InnoDB默认使用REPEATABLE READ而不是READ COMMITTED？** A：主要是为了支持基于语句的复制（Statement-based replication），确保主从数据一致性。

**Q：REPEATABLE READ真的能完全避免幻读吗？** A：在快照读情况下能避免，但在当前读情况下，InnoDB通过Next-Key Lock机制也能避免幻读。

**Q：如何修改事务隔离级别？**

```sql
-- 修改全局级别
SET GLOBAL transaction_isolation = 'READ-COMMITTED';

-- 修改会话级别
SET SESSION transaction_isolation = 'READ-COMMITTED';
```

这个知识点经常与锁机制、MVCC、MySQL复制等内容结合考查，是理解InnoDB存储引擎的基础。