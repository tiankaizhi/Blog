## 写在前面

文章涉及到的 customer 表来源于案例库 sakila，下载地址为 http://downloads.mysql.com/docs/sakila-db.zip

## MySQL 排序方式

+ 通过索引顺序扫描直接返回有序数据

+ 通过对返回数据进行排序，即 FileSort 排序。

所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。FileSort 并不代表通过磁盘文件进行排序，而只是说进行了一个排序操作，至于排序操作是否使用了磁盘文件或临时表取决于 MySQL 服务器对排序参数的设置和需要排序数据的大小。

## EXPLAIN 排序分析

customer DDL

```sql
CREATE TABLE `customer` (
  `customer_id` smallint unsigned NOT NULL AUTO_INCREMENT,
  `store_id` tinyint unsigned NOT NULL,
  `first_name` varchar(45) NOT NULL,
  `last_name` varchar(45) NOT NULL,
  `email` varchar(50) DEFAULT NULL,
  `address_id` smallint unsigned NOT NULL,
  `active` tinyint(1) NOT NULL DEFAULT '1',
  `create_date` datetime NOT NULL,
  `last_update` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`customer_id`),
  KEY `idx_fk_store_id` (`store_id`),
  KEY `idx_fk_address_id` (`address_id`),
  KEY `idx_last_name` (`last_name`),
  KEY `idx_storeid_email` (`store_id`,`email`),
  CONSTRAINT `fk_customer_address` FOREIGN KEY (`address_id`) REFERENCES `address` (`address_id`) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT `fk_customer_store` FOREIGN KEY (`store_id`) REFERENCES `store` (`store_id`) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=600 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE DEFINER=`root`@`%` TRIGGER `customer_create_date` BEFORE INSERT ON `customer` FOR EACH ROW SET NEW.create_date = NOW();
```

---

返回所有用户记录，根据 customer_id 进行排序。因为 customer_id 是主键，记录都是按照主键排好序的，所以无需进行额外的排序操作，直接返回所有数据即可。

```sql
EXPLAIN SELECT * FROM customer ORDER BY customer_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155451122-1613460412.png)

---

由于默认是根据 customer_id 进行排序的，相对于上面来说，这里需要全表扫描，然后根据 store_id 对全表扫描结果进行排序，因此这里用到了 FileSort 排序。

```sql
EXPLAIN SELECT * FROM customer ORDER BY store_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155517932-108466437.png)

---

store_id , email 这两个字段是有联合索引 idx_storeid_email 的，只查询 store_id 和 email 两个字段，直接通过联合索引所在的 B+ 树返回查询数据（该索引树先根据 store_id 字段先进行排序，然后再根据 email 字段排序好的），所以这里的查询结果就是排序好的。

```sql
EXPLAIN SELECT store_id , email FROM customer ORDER BY store_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155534307-1507703550.png)

---

和上面不同的是，排序的字段是 email，导致 FileSort 排序的原因是联合索引 idx_storeid_email 的是先根据 store_id 字段先进行排序，然后再根据 email 字段排序好的，如果直接使用 email 排序，则无法使用 idx_storeid_email 索引树排好的顺序。

```sql
EXPLAIN SELECT store_id , email FROM customer ORDER BY email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155547371-1164747859.png)

---

ORDER BY 使用联合索引进行排序

```sql
EXPLAIN SELECT store_id , email FROM customer ORDER BY store_id , email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155558784-1066268844.png)

---

ORDER BY 的字段混用 ASC 和 DESC 会导致使用 FileSort 排序。

```sql
EXPLAIN SELECT store_id , email FROM customer ORDER BY store_id ASC , email DESC;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155610444-855600068.png)

---

固定 store_id = 1 情况下对 email 字段进行排序，使用 idx_storeid_email 索引即可

```sql
EXPLAIN SELECT store_id , email FROM customer WHERE store_id  = 1 ORDER BY email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155621548-856717287.png)

---

where 条件先进行 store_id 范围查询导致 ORDER BY email 字段无法使用 idx_storeid_email 索引进行排序。

```sql
EXPLAIN SELECT store_id , email FROM customer WHERE store_id  >= 1 AND store_id <= 3 ORDER BY email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155632449-1327608171.png)

---

### ORDER BY 可能出现 FileSort 的几种情况：
1. order by 字段混用 ASC 和 DESC 排序方式。
2. SELECT * 时，如果 order by 排序字段不是主键可能导致 FileSort。
3. 联合索引情况下，order by 多字段排序的字段左右顺序和联合索引的字段左右顺序不一致导致 FileSort。
4. 联合索引情况下，where 字段和 order by 字段的左右顺序和联合索引字段左右顺序或者 where 字段出现范围查询都可能导致 FileSort。


## FileSort 优化

通过创建合适的索引可以减少 FileSort 的出现，但是对于某些情况下，条件限制不能让 FileSort 彻底消失，那就需要对 FileSort 进行优化。对于 FileSort，MySQL 有两种排序算法。

+ 两次扫描算法（Two Passes）：首先根据条件取出排序字段和行指针，之后在排序区 sort buffer 中排序。如果排序区 sort buffer 不够，则在临时表 Temporary Table 中存储排序结果。完成排序后根据行指针回表读取完整记录。该算法是 MySQL 4.1 之前采用的算法，需要两次访问数据，第一次获取排序字段和行指针信息，第二次根据行指针获取完整记录，尤其是第二次读取操作可能导致大量随机 IO 操作；有点是排序的时候内存开销比较小。

+ 一次扫描算法（Single Passes）：一次性取出满足条件的行的所有字段，然后在排序区 sort buffer 中排序后直接输出结果集。排序的时候内存开销比较大，但是排序效率比两次扫描算法要高。

MySQL 通过比较系统变量 max_length_for_sort_data 的大小和 Query 语句取出的字段总大小来判断使用哪种排序算法。如果 max_length_for_sort_data 设置足够大，那么会使用一次扫描算法；否则使用两次扫描算法。适当加大系统变量 max_length_for_sort_data 的值，能够让 MySQL 选择更加优化的 FileSort 排序算法。当然，假如 max_length_for_sort_data 设置过大，会造成 CPU 利用率过低和磁盘 IO 过高。

适当加大 sort_buffer_size 排序区，尽量让排序在内存中完成，而不是通过创建临时表放在文件中进行；当然也不能无限加大 sort_buffer_size 排序区，因为 sort_buffer_size 参数是每个线程独占的，设置过大会导致服务器 SWAP 严重，要考虑数据库活动连接数和服务器内存的大小来适当设置排序区。

尽量只使用必要的字段，SELECT 具体的字段名称，而不是 SELECT * 选择所有字段，这样可以减少排序区的使用，提高 SQL 性能。

#### 参考

《深入浅出 MySQL 数据库开发、优化与管理维护第 2 版》

《MySQL 实战 45 讲》

## 最后

如果大家想要实时关注我更新的文章以及我分享的干货的话，可以关注我的公众号 **<font color="#de2c58">我们都是小白鼠</font>**。

![](https://img2020.cnblogs.com/blog/1326851/202003/1326851-20200307235900287-613114059.png)
