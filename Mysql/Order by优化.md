## 写在前面

文章涉及到的 customer 表来源于案例库 sakila，下载地址为 ```http://downloads.mysql.com/docs/sakila-db.zip```，另外文章演示的 Demo 基于 MySQL Community Server ```8.0.19``` 版本。

## MySQL 排序方式基本可以分为两种

+ 运用索引天然排序的特征直接返回排好序的数据

+ 通过对返回数据进行排序，即 FileSort 排序

所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。FileSort 并不代表通过磁盘文件进行排序，而只是说进行了一个排序操作，至于 <i>**排序操作是否使用了磁盘文件取决于 MySQL 服务器对排序参数的设置和需要排序数据的大小**</i>。

## EXPLAIN 排序分析

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

### 主键索引

根据 customer_id 进行排序。因为 customer_id 是主键，记录都是按照主键排好序的，所以无需进行额外的排序操作，直接返回所有数据即可。

```sql
EXPLAIN SELECT * FROM customer ORDER BY customer_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155451122-1613460412.png)

### 普通索引

由于整张表的所有记录默认是根据 customer_id 排好序的，当然 store_id 索引树也是排好序的，但是仅仅是对 store_id 这个字段来说是排好序的，这里的需求是整张表按 store_id 进行排序，所以涉及到了 FileSort 排序。

```sql
EXPLAIN SELECT * FROM customer ORDER BY store_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155517932-108466437.png)

```sql
EXPLAIN SELECT store_id FROM customer ORDER BY store_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202009/1326851-20200906145257741-1283764525.png)

### 联合索引

store_id , email 这两个字段是有联合索引 idx_storeid_email 的，只查询 store_id 和 email 两个字段，直接通过联合索引所在的 B+ 树返回查询数据（该索引树先根据 store_id 字段先进行排序，然后再根据 email 字段排序好的），所以这里的查询结果就是排序好的。

```sql
EXPLAIN SELECT store_id, email FROM customer ORDER BY store_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155534307-1507703550.png)

和上面不同的是，排序的字段是 email，导致 FileSort 排序的原因是联合索引 idx_storeid_email 的是先根据 store_id 字段先进行排序，然后再根据 email 字段排序好的，如果直接使用 email 排序，则无法使用 idx_storeid_email 索引树排好的顺序，需要先从 idx_storeid_email 索引树上将 store_id 和 email 这两个字段查出来然后再根据 email 字段进行排序。

```sql
EXPLAIN SELECT store_id, email FROM customer ORDER BY email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155547371-1164747859.png)

按照联合索引顺序多字段排序

```sql
EXPLAIN SELECT store_id, email FROM customer ORDER BY store_id, email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155558784-1066268844.png)

不按照联合索引多字段顺序进行排序

```sql
EXPLAIN SELECT store_id, email FROM customer ORDER BY email, store_id;
```

![](https://img2020.cnblogs.com/blog/1326851/202009/1326851-20200906145337134-2056167140.png)

这里注意，和 SELECT 的字段顺序没有关系

```sql
EXPLAIN SELECT email, store_id FROM customer ORDER BY store_id, email;
```

![](https://img2020.cnblogs.com/blog/1326851/202009/1326851-20200906145410016-1053443924.png)

idx_storeid_email 索引是按照 store_id 和 email 升序进行排序的，这里如果 email 按照降序来排序，那前面 store_id 升序排序可以继续使用索引排好序，后面 email 降序是要进行再排序的。

```sql
EXPLAIN SELECT store_id, email FROM customer ORDER BY store_id ASC, email DESC;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155610444-855600068.png)

固定 store_id = 1 情况下对 email 字段进行排序，使用 idx_storeid_email 索引即可

```sql
EXPLAIN SELECT store_id, email FROM customer WHERE store_id  = 1 ORDER BY email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155621548-856717287.png)

where 条件先进行 store_id 范围查询导致 ORDER BY email 字段无法使用 idx_storeid_email 索引进行排序。

```sql
EXPLAIN SELECT store_id, email FROM customer WHERE store_id  >= 1 AND store_id <= 3 ORDER BY email;
```

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200528155632449-1327608171.png)

## ORDER BY 可能出现 FileSort 的几种情况

无法直接利用索引树排好序的，基本都会出现 Using FileSort。常见的情况如下：

1、一般全表数据默认只会按照主键进行排好序，所以，如果需要 SELECT * 时，或者 SELECT 的字段没有建立索引时如果不是按照主键进行排序，那么是要再排序的。

2、对于联合索引来说，索引树是按照多个字段的升序排列，如果你 order by 的时候涉及到多字段升序和降序混用，会导致无法利用索引的天然排序，或者是只能利用到一部分，另一部分需要再排序。

3、联合索引情况下，order by 多字段排序的字段左右顺序和联合索引的字段左右顺序不一致导致 FileSort。

4、联合索引情况下，where 字段和 order by 字段的左右顺序和联合索引字段左右顺序或者 where 字段出现范围查询都可能导致 FileSort。

## 排序的优化

从上面案例我们大致能了解到，要想优化排序，其实就是尽量使用索引排好的序，减少再排序，也就是尽量减少 Using FileSort 的出现。下面我们来了解一下排序的基本原理。

### 全字段排序和 rowid 排序

#### 全字段排序

如果 MySQL 认为内存足够大，会优先选择全字段排序，把需要的字段都放到 ```sort_buffer``` 中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。

这也就体现了 MySQL 的一个设计思想：如果内存够，就要多利用内存，尽量减少磁盘访问。

#### rowid 排序

上面的全字段排序涉及到一个问题，就是如果查询要返回的字段很多的话，那么 ```sort_buffer``` 里面要放的字段数太多，这样内存里能够同时放下的行数很少，比如：符合条件的记录一共 1000 行，内存只能放 500 行，那么还有 500 行就要水平拆分放入多个临时文件进行排序（究竟拆分成多少个临时文件排序，也涉及到相关算法，这里就不深入研究了），所以如果单行很大，这个方法效率不够好。那么，如果 MySQL 认为排序的单行长度太大会怎么做呢？接下来，我来修改一个参数，让 MySQL 采用另外一种算法。

```sql
SET max_length_for_sort_data = 16;  
```

```max_length_for_sort_data```，是 MySQL 中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL 就认为单行太大，要换一个排序算法。


假设现在单行超过了 ```max_length_for_sort_data```，那么就会采用 rowid 进行排序，首先会将需要排序的字段和主键取出来到内存中进行排序，排好序之后，再根据主键进行回表查出其他字段直接返回给客户端。注意，这里回表查询的数据不会存放到内存中，而是直接返回给客户端。这样就缓解了全字段排序可能导致很多记录磁盘排序的问题。但是 rowid 排序同样也引入了另一个问题，那就是回表，如果回表过多也会导致性能下降。

> MySQL ```8.0.19``` 版本默认 ```max_length_for_sort_data``` 大小为 ```4096 字节```，即 ```4kb```；默认 ```sort_buffer_size``` 大小为 ```262144 字节```，即 ```256kb```。

## 总结

MySQL 通过 ```max_length_for_sort_data``` 的大小单行所有列的总大小来判断使用哪种排序算法。如果 ```max_length_for_sort_data``` 设置足够大，那么会使用一次扫描算法；但是，一次性性将很多行数据都加载到
```sort_buffer``` 中，如果 ```sort_buffer``` 放不下，就会导致大量记录会使用到磁盘文件排序。此时磁盘负载就会过高，内存中排序的记录就会变少，相应的 CPU 利用率就会过低。如果设置的很小，则会使用 rowid 排序，也会导致回表过多，性能过差。

比较权衡的做法是，适当加大 ```sort_buffer_size``` 排序区，尽量让排序在内存中完成，而不是在磁盘文件中进行排序；当然也不能无限加大 ```sort_buffer_size``` 排序区，因为 ```sort_buffer_size``` 参数是每个线程独占的，设置过大可能会导致服务器 SWAP 严重，**要考虑数据库活跃连接数和服务器内存的大小来适当设置排序区**。

尽量只使用必要的字段，SELECT 具体的字段名称，而不是 SELECT * 选择所有字段，这样可以减少排序区的使用，提高 SQL 性能。

#### 参考

《深入浅出 MySQL 数据库开发、优化与管理维护第 2 版》

《MySQL 实战 45 讲》
