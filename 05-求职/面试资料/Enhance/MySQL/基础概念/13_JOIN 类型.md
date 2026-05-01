## MySQL Join操作的主要类型

### 1. INNER JOIN（内连接）

**最常用的连接类型**，只返回两个表中都存在匹配记录的行。

```sql
SELECT u.name, o.order_id 
FROM users u 
INNER JOIN orders o ON u.user_id = o.user_id;
```

### 2. LEFT JOIN（左外连接）

返回左表的所有记录，即使右表中没有匹配的记录。右表没有匹配时显示NULL。

```sql
SELECT u.name, o.order_id 
FROM users u 
LEFT JOIN orders o ON u.user_id = o.user_id;
```

### 3. RIGHT JOIN（右外连接）

返回右表的所有记录，即使左表中没有匹配的记录。左表没有匹配时显示NULL。

```sql
SELECT u.name, o.order_id 
FROM users u 
RIGHT JOIN orders o ON u.user_id = o.user_id;
```

### 4. FULL OUTER JOIN（全外连接）

**注意：MySQL不直接支持FULL OUTER JOIN**，但可以通过UNION实现：

```sql
SELECT u.name, o.order_id 
FROM users u LEFT JOIN orders o ON u.user_id = o.user_id
UNION
SELECT u.name, o.order_id 
FROM users u RIGHT JOIN orders o ON u.user_id = o.user_id;
```

### 5. CROSS JOIN（交叉连接/笛卡尔积）

返回两个表的笛卡尔积，每个左表记录与每个右表记录组合。

```sql
SELECT u.name, p.product_name 
FROM users u 
CROSS JOIN products p;
```

### 6. SELF JOIN（自连接）

表与自身进行连接，常用于处理层级关系数据。

```sql
SELECT e1.name as employee, e2.name as manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;
```
