http://www.cnblogs.com/dato/p/7049343.html

### 浅谈 DML、DDL、DCL的区别

一、DML

- DML（data manipulation language）数据操纵语言：

　　　　就是我们最经常用到的 SELECT、UPDATE、INSERT、DELETE。 主要用来对数据库的数据进行一些操作。

```sql
SELECT 列名称 FROM 表名称
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
DELETE FROM 表名称 WHERE 列名称 = 值
```

二、DDL

- DDL（data definition language）数据库定义语言：

　　　　其实就是我们在创建表的时候用到的一些sql，比如说：CREATE、ALTER、DROP等。DDL主要是用在**定义或改变表的结构，数据类型，表之间的链接和约束等初始化工作上**

```sql
CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)

ALTER TABLE table_name
ALTER COLUMN column_name datatype

DROP TABLE 表名称
DROP DATABASE 数据库名称
```

三、DCL

- DCL（Data Control Language）数据库控制语言：

　　　　是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。这个比较少用到。

在公司呢一般情况下我们用到的是DDL、DML这两种。
