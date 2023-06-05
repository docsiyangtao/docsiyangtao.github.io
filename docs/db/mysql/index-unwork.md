偶然间组长发现在我在where中使用了函数，被臭骂了一顿，因此总结下几种索引失效的情况，整理时无意间发现了MySQL会根据“成本”选择全表扫描还是使用索引，收获不小~                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     

## 准备

user表，插入数据若干条

```mysql
CREATE TABLE `user` (
  `uid` int(8) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `age` int(8) DEFAULT NULL,
  `phone` int(11) DEFAULT NULL,
  `created_at` datetime DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



## 索引失效分析

### 不满足最左匹配原则

创建联合索引 **idx_name_age_phone**

```mysql
alter table user add index idx_name_age_phone(name, age, phone);
```

**正常情况，三个字段都走了索引**

```mysql
mysql> explain select * from user where name='x' and age=12 and phone=123\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: idx_name_age_phone
          key: idx_name_age_phone
      key_len: 72
          ref: const,const,const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

**联合索引中没有取到第一个列，导致索引失效，使用了全表扫描**

```mysql
explain select * from user where age=100\G

*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

### 跳过索引中间列

**联合索引跳过了中间某个列，导致部分字段没有走索引**

```mysql
mysql> explain select * from user where name='x' and phone=123\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: idx_name_age_phone
          key: idx_name_age_phone
      key_len: 62
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

### 使用函数或进行计算

**给列created_at添加索引 idx_created_at**

```mysql
alter table user add index idx_created_at(created_at);
```

```mysql
mysql> explain select * from user where datediff(now(), created_at) > 1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

解决：修改SQL、把函数计算放在与索引字段分开

```mysql
mysql> explain select * from user where created_at < date_add(now(), interval -1 day)\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_created_at
          key: idx_created_at
      key_len: 6
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

MySQL 8.0开始可以通过添加函数索引解决，注：**仅支持已知变量的函数和列**，如函数中包含不确定的变量，则不能添加，如

```mysql
mysql> alter table user add key idx_fun_created_at((datediff(now(), created_at)));
ERROR 3758 (HY000): Expression of functional index 'idx_fun_created_at' contains a disallowed function.
```

对于不确定的now()，是不能添加函数索引的，正确的添加：datediff中**指定变量和列**，指明变量为'2022-01-01 00:00:00'和列 created_at

```mysql
mysql> alter table user add key idx_fun_created_at((datediff('2022-01-01 00:00:00', created_at)));
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain select * from user where datediff('2022-01-01 00:00:00', created_at) > 1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_fun_created_at
          key: idx_fun_created_at
      key_len: 5
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

这样**再使用datediff('2022-01-01 00:00:00', created_at)函数，就可以走索引了**

对于计算也是同样如此，添加计算式索引，使用计算式做条件也可以走索引，如：

```mysql
mysql> alter table user add key idx_fun_uid((age + 1));
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain select * from user where age+1=1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: idx_fun_uid
          key: idx_fun_uid
      key_len: 5
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

### 字段类型不同

如数据库中的varchar类型字段，在查询时使用了int类型

```mysql
mysql> alter table user add index idx_name(name);
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain select * from user where name=1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: idx_name
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 3 warnings (0.00 sec)
```

特殊：如果数据库中是int类型，但是查询时使用的是varchar类型，仍然会走索引，因为MySQL会对int类型列的传参进行隐式转换，**即将where中的varchar类型转为int类型**，如uid为int类型，且是主键，此时传参为varchar类型

```mysql
mysql> explain select * from user where uid='1'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

### 模糊查询%写在左边

对于拥有索引的列name，进行模糊查询，正常（%不写在最左边）情况，可以走索引，如：

```mysql
mysql> explain select * from user where name like '1%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_name
          key: idx_name
      key_len: 62
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

但是%写在最左边，不做任何优化，则会全表扫描，如：

```mysql
mysql> explain select * from user where name like '%1'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)


```

如果一定要将%写在最左边，则优化策略为：为查询的列添加索引，让其从索引上取数据，这样会比全表扫描更快，也就是常说的 **索引覆盖**。如：查询uid和name（这两列都是有索引的列）

```mysql
mysql> explain select name from user where name like '%1'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index
possible_keys: NULL
          key: idx_name
      key_len: 62
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where; Using index
1 row in set, 1 warning (0.01 sec)
```

可看到使用了索引 idx_name，Extra为Using index

### or 连接

or连接也会导致索引失效，走全表扫描，即使是两个都有各自的索引，如uid和age

```mysql
mysql> explain select uid,phone from user where uid=1 or age=2\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: PRIMARY,idx_age,idx_id_name
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

解决：

1. 索引覆盖

   ```mysql
   mysql> alter table user add index idx_uid_phone(uid, phone);
   Query OK, 0 rows affected (0.03 sec)
   Records: 0  Duplicates: 0  Warnings: 0
   
   mysql> explain select uid,phone from user where uid=1 or phone=188\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: user
      partitions: NULL
            type: index
   possible_keys: PRIMARY,idx_id_name,idx_uid_phone
             key: idx_uid_phone
         key_len: 9
             ref: NULL
            rows: 1
        filtered: 100.00
           Extra: Using where; Using index
   1 row in set, 1 warning (0.00 sec)
   ```

2. union all

   先为or连接的列加索引，再使用union all代替or连接

   ```mysql
   mysql> alter table user drop index idx_uid_phone;
   Query OK, 0 rows affected (0.02 sec)
   Records: 0  Duplicates: 0  Warnings: 0
   
   mysql> alter table user add index idx_phone(phone);
   Query OK, 0 rows affected (0.02 sec)
   Records: 0  Duplicates: 0  Warnings: 0
   
   mysql> explain select * from user where uid=1 union all select * from user where phone=188\G
   *************************** 1. row ***************************
              id: 1
     select_type: PRIMARY
           table: user
      partitions: NULL
            type: const
   possible_keys: PRIMARY,idx_id_name
             key: PRIMARY
         key_len: 4
             ref: const
            rows: 1
        filtered: 100.00
           Extra: NULL
   *************************** 2. row ***************************
              id: 2
     select_type: UNION
           table: user
      partitions: NULL
            type: ref
   possible_keys: idx_phone
             key: idx_phone
         key_len: 5
             ref: const
            rows: 1
        filtered: 100.00
           Extra: NULL
   2 rows in set, 1 warning (0.00 sec)
   ```

### not exists

```mysql
mysql> explain select * from user t1 where not exists (select 1 from user t2 where t1.uid = t2.uid and t2.age>25)\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: t2
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY,idx_age,idx_id_name
          key: PRIMARY
      key_len: 4
          ref: mydb.t1.uid
         rows: 1
     filtered: 100.00
        Extra: Using where; Not exists
2 rows in set, 2 warnings (0.00 sec)
```

### is not null、!=、<>

首先要明确一个点：**走索引不一定不不走索引快**。决定是否使用索引的因素是：**成本**。

> 成本组成主要有两个方面：
>
> - 读取二级索引记录的成本
> - 将二级索引记录执行回表操作，也就是到聚簇索引中找到完整的用户记录的操作所付出的成本。
>
> 很显然，要扫描的二级索引记录条数越多，那么需要执行的回表操作的次数也就越多，达到了某个比例时，使用二级索引执行查询的成本也就超过了全表扫描的成本（举一个极端的例子，比方说要扫描的全部的二级索引记录，那就要对每条记录执行一遍回表操作，自然不如直接扫描聚簇索引来的快）。
>
> 所以MySQL优化器在真正执行查询之前，对于每个可能使用到的索引来说，都会预先计算一下需要扫描的二级索引记录的数量
>
> ——《[MySQL中IS NULL、IS NOT NULL、!=不能用索引？胡扯](https://zhuanlan.zhihu.com/p/78761196)》

如，在age列新建索引，age为null的数据量占比太大了，而且达到了3万多条，此时查询成本：走索引 > 全表扫描，因此MySQL选择了全表扫描

```mysql
mysql> explain select * from user where age is not null\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: idx_age
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 33664
     filtered: 50.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

而在数据量小的时候，却使用了索引，如：

```mysql
mysql> explain select * from user where age is null\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: idx_age
          key: idx_age
      key_len: 5
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

### ORDER BY的坑

创建复合索引**idx_uid_age_phone**

```mysql
mysql> alter table user add index idx_uid_age_phone(uid,age,phone);
Query OK, 0 rows affected (0.16 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除之前建立的索引，创建复合索引

```mysql
alter table user add index idx_uid_age_phone(uid,age,phone);
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

还是“成本”问题，不使用where或limit的情况不走索引：

```mysql
mysql> explain select * from user order by uid,age,phone\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 34578
     filtered: 100.00
        Extra: Using filesort
1 row in set, 1 warning (0.00 sec)
```

使用了where或limit，小数据量下走了索引：

```mysql
mysql> explain select * from user where uid < 20 order by uid,age,phone limit 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: PRIMARY,idx_uid_age_phone
          key: idx_uid_age_phone
      key_len: 4
          ref: NULL
         rows: 19
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

排序字段不满足最左前缀匹配原则导致索引失效

```mysql
mysql> explain select * from user order by age,uid,phone limit 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 34578
     filtered: 100.00
        Extra: Using filesort
1 row in set, 1 warning (0.00 sec)
```

排序字段断层导致索引失效，全表扫描

```mysql
mysql> explain select * from user order by uid,phone limit 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 34578
     filtered: 100.00
        Extra: Using filesort
1 row in set, 1 warning (0.00 sec)
```

排序方式没有统一，部分降序部分升序，导致索引失效

```mysql
mysql> explain select * from user order by uid asc,age desc,phone limit 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 34578
     filtered: 100.00
        Extra: Using filesort
1 row in set, 1 warning (0.00 sec)
```



<center>【END】</center>

