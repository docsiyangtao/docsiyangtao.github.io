## 计算列

某一列的值是通过别的列计算得来的，如a列值为1、b列值为2，c列不需要手动插入，定义a+b的结果为c的值，那么c就是计算列，是通过别的列计算得来的。

### 创建

```mysql
create table mytb(
	id int primary key auto_increment,
	a int,
	b int,
	c int generated always as (a+b) virtual
)engine = Innodb default charset = utf8mb4;

insert into mytb(a,b) values(1,2);

select * from mytb;
```

```bash
+----+------+------+------+
| id | a    | b    | c    |
+----+------+------+------+
|  1 |    1 |    2 |    3 |
+----+------+------+------+
1 row in set (0.00 sec)
```

### 修改

```mysql
alter table mytb add d int generated always as (a+b+c) virtual;
```

```bash
+----+------+------+------+------+
| id | a    | b    | c    | d    |
+----+------+------+------+------+
|  1 |    1 |    2 |    3 |    6 |
+----+------+------+------+------+
1 row in set (0.00 sec)
```



## 自增变量持久化

8.0之前，MySQL重启后，会重置auto_increment = max(primary key) + 1，因为对于自增的分配规则是InnoDB内部的一个计数器决定，且该计数器在内存中维护，8.0之后这个计数器被持久化到重做日志中，如果数据库重启，InnoDB会根据重做日志中的信息来初始化计数器的内存值。
