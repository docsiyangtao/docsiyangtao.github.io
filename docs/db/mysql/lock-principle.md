# MySQL锁机制

## 表锁

### 优点

* 开销小
* 无死锁

### 缺点

* 粒度大
* 锁冲突概率高
* 并发度低

### 读锁

又称为`共享锁`

创建两张测试表`test_1`和`test_2`

```mysql
create table test_1(
    ID bigint not null primary key auto_increment,
    COL_1 varchar(50) not null
) engine = innodb
  auto_increment = 1
  default charset = utf8mb4 comment = '测试表_1';

create table test_2(
    ID bigint not null primary key auto_increment,
    COL_1 varchar(50) not null
) engine = innodb
  auto_increment = 1
  default charset = utf8mb4 comment = '测试表_2';
```

演示开始

![image-20220912235850382](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220912235850382.png)

![image-20220913000010229](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913000010229.png)

![image-20220913000207741](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913000207741.png)

![image-20220913000352109](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913000352109.png)

### 写锁

![image-20220913003638346](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913003638346.png)

![image-20220913003752612](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913003752612.png)

![image-20220913003951799](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913003951799.png)

![image-20220913004133502](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913004133502.png)

![image-20220913004236582](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220913004236582.png)

### 其他命令

```shell
show status like 'table%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Table_locks_immediate      | 193   |
| Table_locks_waited         | 0     |
| Table_open_cache_hits      | 4     |
| Table_open_cache_misses    | 2     |
| Table_open_cache_overflows | 0     |
+----------------------------+-------+
5 rows in set

# Table_locks_immediate：产生表级锁定的次数，表示可以立即获得锁的查询次数
# Table_locks_waited：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），
					  此值高则说明存在着较严重的表级锁争用情况
```

### 总结

1. session-1加了读锁，则：session-1、session-2都可以读；session-1不可写，session-2写阻塞；session-1不能读其他，session-2可读其他。
2. session-1加了写锁，则：session-1可读，session-2读阻塞；session-1可写，session-2写阻塞；session-1不能读其他，session-2可读其他。
3. MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎，因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。



## 行锁

InnoDb存储引擎默认行锁

### 回顾事务

#### 事件

* 脏读：一个事务读取到了另一个事务未提交的数据
* 不可重复读：一个事务多次读取同一条记录，读取的结果不一致（读取到另一个事务已提交的数据）
* 幻读（虚读）：一个事务多次查询的数据条数不一致（读取到另一个事务增加或删除的数据）

#### 隔离级别

* read uncommitted（读未提交）：不做隔离，具有脏读、不可重复读、幻读等问题
* read committed（读提交）：可避免脏读、不可避免不可重复读、虚读等问题
* repeatable read（可重复读）：可以避免脏读、不可重复读、虚读，是mysql的默认隔离级别
* serializable（可串行化）：每个数据加上锁，最高隔离级别，但会出现超时现象，很少应用

### innodb

InnoDb在默认状态下就已经解决了脏读、不可重复读、幻读这3个问题

演示：

![image-20220915235045387](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220915235045387.png)

![image-20220915235506639](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220915235506639.png)

![image-20220916000002152](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220916000002152.png)

### MVCC

多版本并发控制（MCCC）（快照都 / 一致性读）

数据库依靠保存数据快照实现了多版本并发控制。在innodb中每一行都冗余了两个字段，一个是行的创建版本，一个是行的删除（过期）版本，具体的版本号存在于information_schema.INNODB_TRX 表中。每次开启事务，读取数据时都会取创建版本小于当前事务版本以及删除版本大于当前版本的数据。

普通的select就是快照读，其他事务的增加与删除对于当前事务来说是不可见的。

### 行锁演示

![image-20220916000416293](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220916000416293.png)

![image-20220915234205508](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220915234205508.png)

显式声明加行锁：for update

![image-20220916000908810](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220916000908810.png)

### 行锁失效变表锁

索引失效会导致行锁变表锁，演示如下：

![image-20220918233336043](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220918233336043.png)

结论：本来更新是行锁，但是session-1使用的where条件没走上索引，所以扫描为全表扫描，给test_1表的所有行都加上了锁，所以session-2中更新不同的行数据仍然会被阻塞。

### 其他参数

```bash
show status like 'innodb_row_lock%';
+-------------------------------+--------+
| Variable_name                 | Value  |
+-------------------------------+--------+
| Innodb_row_lock_current_waits | 0      |
| Innodb_row_lock_time          | 116128 |
| Innodb_row_lock_time_avg      | 19354  |
| Innodb_row_lock_time_max      | 50436  |
| Innodb_row_lock_waits         | 6      |
+-------------------------------+--------+
5 rows in set

# Innodb_row_lock_current_waits:当前正在等待锁定的数量;
# Innodb_row_lock_time:从系统启动到现在锁定总时间长度;
# Innodb_row_lock_time_avg:每次等待所花平均时间;
# Innodb_row_lock_time_max:从系统启动到现在等待最常的一次所花的时间;
# Innodb_row_lock_waits:系统启动后到现在总共等待的次数;
```

### 总结

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
- 合理设计索引，尽量缩小锁的范围
- 尽可能较少检索条件，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可能低级别事务隔离



## 间隙锁

演示

![image-20220918234103844](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220918234103844.png)

session-1更新锁的范围是ID<4，即使这个范围没有ID = 2的数据，但同样会对这个间隙进行上锁，导致session-2插入ID = 2的数据失败。
