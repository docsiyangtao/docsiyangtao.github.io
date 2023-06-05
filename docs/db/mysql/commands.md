## DDL

### 数据库

```mysql
-- 创建数据库
create database 数据库名 [character set 编码] [collate 校对规则] [if not exists 数据库名];

-- 查看字符集和校对规则
show character set;

-- 查看所有数据库
show databases;

-- 查看数据库的创建信息
show create database 数据库名;

-- 使用数据库
use 数据库名;

-- 查看正在使用的数据库
select database();

-- 修改数据库
alter database 数据库名 default character set 编码方式 collate 校对规则;

-- 删除数据库
drop database 数据库名 [if exists 数据库名];
```

* 编码：utf8mb4、utf8 、GBK等
* 校对规则：utf8_general_ci（不区分大小写）、utf8_general_cs（区分大小写）、utf8_bin（二进制）等

### 数据表

```mysql
-- 创建数据库
create table 表名(
	字段名 数据类型 完整性约束条件 [character set 字符集名称] [collate 比较规则],
	字段名 数据类型 完整性约束条件,
	字段名 数据类型 完整性约束条件
)engine=存储引擎 default charset=编码方式;

-- 查看数据表
show create table 表名;
desc 表名;

-- 查看表的所有列信息
show columns from 表名;

-- 修改表名
alter table 表名 rename 新表名;

-- 修改字段名
alter table 表名 change 旧字段名 新字段名 新数据类型;

-- 修改字段数据类型
alter table 表名 modify 字段名 数据类型;

-- 添加字段
alter table 表名 add 字段名 数据类型 [约束条件] [first/after已存在字段名];

-- 删除字段
alter table 表名 drop 字段名;

-- 修改字段排列位置
alter table 表名 modify 字段名1 数据类型 first/after 字段名2;

-- 添加外键
alter table 表名 add constraint [外键名] foreign key(主表字段) references 外表表名(字段名);

-- 删除外键
alter table 表名 drop foreign key 外键名;

-- 查看表状态
show table status like '表名';
show table status from 数据库名 like '表名';

-- 清空表
truncate table 表名;

-- 查看表的约束
select * from information_schema.table_constraints where table_name = '表名称';
```

* 数据类型
  * 整数型：TINYINT、SMALLINT、MEDIUMINT、**`INT`**、BIGINT
  * 浮点型：FLOAT、DOUBLE
  * 定点数：**`DECIMAL`**
  * 逻辑型：BIT
  * 日期型：YEAR、TIME、DATE、DATETIME、**`TIMESTAMP`**
  * 文本型：CHAR、**`VARCHAR`**、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT
  * 枚举型：ENUM
  * 集合型：SET
  * 二进制字符串：BINARY、VARBINARY、TINYBLOG、BLOG、MEDIUMBLOB、LONGBLOB
  * 其他...
* 约束
  * primary key：主键约束
  * foreign key：外键约束
  * not null：非空约束
  * unique：唯一性约束
  * default：默认值约束
  * auto_increment：字段值自增
  * unsigned：无符号约束
  * character set：字符集约束

### 视图

```mysql
-- 创建/修改
create [or replace]
[algorithm = {undefined | merge | temptable}]
view 视图名称 [(字段列表)]
as 查询语句
[with [cascaded|local] check option];

-- 查看
show tables;
desc|describe 视图名称;
show table status like '视图名称';
show create view 视图名称;

-- 修改
alter view 视图名称 as 查询语句;

-- 删除
drop view if exists 视图名称 [, 视图名称2, ...];
```



## DML

```mysql
-- 添加数据
insert into 表名 values(值1, 值2, ......);
insert into 表名(字段1,字段2,...)values(值1,值2,...);
insert into 表名 set 字段名1=值1[,字段名2=值2,...];
insert into 表名[(字段名1,字段名2,...)]values(值1,值2,...),(值1,值2,...)...;

-- 更新数据
update 表名 set 字段名1=值1[,字段名2=值2,...] [where 条件表达式];

-- 删除数据
delete from 表名 [where 条件表达式];
truncate [table] 表名;
```



## DQL

```mysql
-- 简单查询
select * from 表名;

-- 按条件查询
select * from 表名 where 条件表达式 [and 条件表达式];
select * from 表名 where 条件表达式 [or 条件表达式];
select * from 表名 where 字段名 [not] in(元素1,元素2,......);
select * from 表名 where 字段名 [not] between 值1 and 值2;
select * from 表名 where 字段名 is [not] null;
select * from 表名 where 字段名 [not] like '匹配字符串' [escape '特殊字符'];
select * from 表名 where 字段名 regexp '正则表达式';

-- 去重查询
select distinct 字段名 from 表名;

-- 排序查询
select 字段名1,字段名2,... from 表名 order by 字段名1 [asc|desc],字段名2 [asc|desc]...;

-- 分组查询
select 字段名1,字段名2,... from 表名 group by 字段名1, 字段名2, ... [having 条件表达式] [with rollup];

-- 限制查询
select 字段名1,字段名2,... from 表名 limit [位置偏移量, ] 记录数;

-- 取别名
select * from 表名 [as] 别名;

-- 交叉连接查询
select * from 表1, 表2 where 表1.字段 = 表2.字段 [and 条件];
select * from 表1 [[cross] join] 表2 where 表1.字段 = 表2.字段 [and 条件];

-- 内连接查询
select * from 表1 [inner] join 表2 on 表1.关系字段 = 表2.关系字段;

-- 外连接查询
select * from 表1 left | right | [outer] join 表2 on 表1.关系字段 = 表2.关系字段 where 条件;

-- 自然连接
select * from 表1 natural join 表2;

-- USING连接
select * from 表1 join 表2 using(关系字段);

-- 合并查询
select 字段 from 表1 UNION [ALL] select 字段 from 表2;

-- 复合查询
select * from 表1 where x in (select x from 表2);
select * from 表1 where x = (select x from 表2);
select * from 表1 where x > | < any(select x from 表2);
select * from 表1 where x > | < all(select x from 表2);
select * from 表1 where exists(select x from 表2);

-- 总
select distinct ... from ... join ... on ... where ... group by ... having ... order by ... limit ...;
```



## TCL

```mysql
-- 开启事务
start transaction;

-- 提交事务
commit;

-- 取消事务（回滚，只能针对未提交的事务）
rollback;

-- 设置隔离级别
set session transaction isolation level read uncommitted;
set session transaction isolation level read committed;
set session transaction isolation level repeatable read;
set session transaction isolation level serializable;
```



## DCL

```mysql
-- 创建用户
create user '用户名'@'主机名' identified by '密码';
insert into mysql.user(Host,User,Password,ssl_cipher,x509_issuer,x509_subject)values('主机名','用户名',password('密码'),',',',');

-- 删除用户
drop user '用户名'@'主机地址';
delete from mysql.user where user='用户名' and host='主机名';

-- 修改root密码
set password = password('新密码');
update user set password=password('新密码') where user='用户名' and host='主机名';

-- 修改普通用户密码
grant usage on 数据库.表 to '用户名'@'主机名' identified by [password]'new_password';
update mysql.user set password=password('新密码') where user='用户名' and host='主机名';
set password for '用户名'@'主机名'=password('新密码');

-- 普通用户修改密码
set password = password('新密码');

-- 授权
grant 权限 [(columns)][,privileges[(columns)]] on 数据库.表 to '用户名'@'主机' [identified by [密码] [,'用户名'@'主机' [identified by 密码] ... [with grant option ...]

-- 查看权限
show grants for '用户名'@'主机';

-- 回收权限
revoke privileges [columns][,privileges[(columns)]] on 数据库.表 from '用户名'@'主机';
revoke all privileges,grant option from '用户名'@'主机';
```

* 权限：all privileges表示所有权限，其余可通过 **show privileges** 查看
* columns：表示权限作用于某一列，省略不写则权限作用于整个表
* grant option：将自己的权限授予其他用户



## 其他

```bash
# 备份单个数据库
mysqldump -uusername -ppassword dbname>filename.sql

# 备份多个数据库
mysqldump -uusername -ppassword --databases dbname1 dbname2...>filename.sql

# 备份所有数据库
mysqldump -uusername -ppassword --all-databases>filename.sql

# 数据库还原（注意，还原的只是数据，库不能被还原）
mysql -uusername -ppassword dbname<filename.sql

# 修改密码
mysqladmin -uusername -ppassword password 新密码
```

