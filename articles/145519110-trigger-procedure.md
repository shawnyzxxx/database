# 触发器及存储过程

## 文章导读
本篇是数据库课程中**触发器与存储过程**章节的作业，围绕 `mydb7_openlab` 库完成建表、建触发器、建存储过程并做功能测试。触发器能在数据变更时自动执行，存储过程可封装可复用的 SQL 逻辑，二者是数据库自动化与复用的核心。

## 一、作业说明
1. 切换到 `mydb7_openlab` 库
2. 建立表、触发器与存储过程 `s1`、`s2`、`s3`
3. 测试触发器与存储过程

## 二、切换数据库
```sql
USE mydb7_openlab;
```

## 三、创建存储过程
**s1：无参存储过程**
```sql
DELIMITER //
CREATE PROCEDURE s1()
BEGIN
  SELECT * FROM goods;
END //
DELIMITER ;
```

**s2：带输入参数与输出参数**
```sql
DELIMITER //
CREATE PROCEDURE s2(IN p_name VARCHAR(50), OUT p_age INT)
BEGIN
  SELECT COUNT(*) INTO p_age FROM goods WHERE name = p_name;
END //
DELIMITER ;
```

**s3：带输入参数与输出参数**
```sql
DELIMITER //
CREATE PROCEDURE s3(IN p_id INT, OUT p_avg DECIMAL(10,2))
BEGIN
  SELECT AVG(price) INTO p_avg FROM goods WHERE oid = p_id;
END //
DELIMITER ;
```
> `IN` 为输入参数，`OUT` 为输出参数，`DELIMITER` 用于避免存储过程内的 `;` 被提前终止语句。

## 四、创建触发器
触发器在数据变更时自动触发。以「订单变更时联动库存」为例：

**插入订单时减少库存**
```sql
DELIMITER //
CREATE TRIGGER trg_order_insert AFTER INSERT ON orders
FOR EACH ROW
BEGIN
  UPDATE goods SET onum = onum - NEW.onum WHERE gid = NEW.gid;
END //
DELIMITER ;
```

**删除订单时恢复库存**
```sql
DELIMITER //
CREATE TRIGGER trg_order_delete AFTER DELETE ON orders
FOR EACH ROW
BEGIN
  UPDATE goods SET onum = onum + OLD.onum WHERE gid = OLD.gid;
END //
DELIMITER ;
```

**更新订单时更新库存**
```sql
DELIMITER //
CREATE TRIGGER trg_order_update AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
  UPDATE goods SET onum = onum - OLD.onum + NEW.onum WHERE gid = NEW.gid;
END //
DELIMITER ;
```
> 触发器中 `NEW` 表示变更后的新值，`OLD` 表示变更前的旧值。

## 五、测试触发器与存储过程
**1. 测试触发器：插入订单，减少库存**
```sql
INSERT INTO orders (gid, name, price, onum, otime) VALUES ('a0001', '橡皮', 2.5, 10, '2024-01-01');
SELECT * FROM goods WHERE gid = 'a0001';
```

**删除订单，恢复库存**
```sql
DELETE FROM orders WHERE oid = 1;
SELECT * FROM goods WHERE gid = 'a0001';
```

**更新订单，更新库存**
```sql
UPDATE orders SET onum = 5 WHERE oid = 1;
SELECT * FROM goods WHERE gid = 'a0001';
```

**2. 测试存储过程**
```sql
CALL s1();
CALL s2('张三', @age);
SELECT @age;
CALL s3(10, @avg_salary);
SELECT @avg_salary;
```
> 调用带 `OUT` 参数的存储过程时，需要先定义用户变量（如 `@age`、`@avg_salary`）来接收返回值。

## 六、小结
触发器实现「数据变更自动联动」，存储过程实现「SQL 逻辑封装复用」。本次作业通过建表、建触发器、建存储过程与测试，完整走通了数据库自动化控制的流程，是数据库进阶的重要基础。