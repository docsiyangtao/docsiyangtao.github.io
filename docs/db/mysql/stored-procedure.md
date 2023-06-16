# 存储过程

## 存储过程概述

MySQL从5.0开始支持存储过程，能将复杂的SQL逻辑封装在一起，使用时无需关注内部实现，只需简单的调用。

### 思想

一组经过预先编译的SQL语句的封装。

### 执行过程

存储过程预先存储在MySQL服务器上，执行的时候，客户端输入命令和参数调用存储过程，服务器端就可以把存储过程中的一系列SQL语句全部执行。

### 好处

* 一次编译多次使用
* 简化操作，提高SQL的复用性
* 减少操作过程中的失误、提高效率
* 减少网络传输量（只有调用语句和参数）
* 减少SQL暴露，提高数据查询安全性

### 对比视图

视图是虚拟表，通常不对底层数据表直接操作，而存储过程是程序化的SQL，可以直接操作底层数据表，相比于面向集合的操作方式，能够实现一些更复杂的数据处理。

### 分类

* 无参数无返回
* 有参数无返回
* 无参数有返回
* 有参数有返回
* 参数和返回是同一个

### 语法格式

```mysql
create procedure 存储过程名(in|out|inout 参数名 参数类型[, ...])
[characteristics...]
begin
    存储过程体
end

call 存储过程名(...)
```

说明：

1. in、out、inout分别表示输入参数、输出参数、既是输入又是输出参数

2. 形参类型可以是MySQL中的任意类型

3. characteristics表示创建存储过程时指定的对存储过程的约束条件

   ```mysql
   language sql
   # 说明存储过程执行体是由SQL语句组成的，当前系统支持的语言为SQL
   
   | [not] deterministic
   # 存储过程的执行结果是否确定（相同输入是否能得到相同输出）
   
   | {contains sql | no sql | reads sql data | modifies sql data}
   # 使用sql的限制，依次是：包含不读写的sql、不包含sql、含读sql、含写sql，默认为contains sql
   
   | sql security {definer | invoker}
   # 执行当前存储过程的权限，创建者、定义者才能执行，或者拥有当前存储过程访问权限的能执行
   
   | comment 'string'
   # 注释信息，用于描述存储过程
   ```

4. 如果函数体只有一行，begin和end可省略

5. 使用call进行调用，如果有输出参数的，可定义变量进行接收

### 示例

获取省份下的所有城市

```mysql
delimiter //
create procedure getCitiesByProvince(in p_province varchar(20), out p_cities varchar(255))
begin
    select group_concat(c.NAME)
    into p_cities
    from region p
             left join region c on c.PARENT_ID = p.ID
    where p.NAME = p_province;
end //
delimiter ;

call getCitiesByProvince('广东省', @citiesStr);
select @citiesStr;
```

运行结果如下

![image-20230614003716582](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230614003716582.png)

## 存储函数概述

与存储过程类似，MySQL还支持自定义函数，定义完成后，调用方式与调用MySQL预定义的系统函数一致。

语法格式如下

### 语法格式

```mysql
create function 函数名(参数名 参数类型[, ...])
returns 返回值类型
[charateristics...]
begin
	函数体(return ....)
end

select 函数名(...)
```

说明：

1. 存储函数的参数都是in参数（书写时可省略）
2. 函数体中必须包含return value语句
3. charateristics为对函数的约束，取值与创建存储过程相同
4. 如函数体只有一行，begin和end可省略
5. 使用select进行调用
6. 与存储过程不同，创建函数需要定义特征（set global log_bin_trust_function_creators = 1; 可跳过）

### 示例

查询省份下的所有城市

```mysql
delimiter //
create function getCitiesByProvince(p_province varchar(20))
    returns varchar(255)
    deterministic
    contains sql
    reads sql data
    sql security definer
    comment '根据省份获取所有城市'
begin
    return (select group_concat(c.NAME)
            from region p
                     left join region c on c.PARENT_ID = p.ID
            where p.NAME = p_province);
end //
delimiter ;

select getCitiesByProvince('广东省');
```

## 存储过程与存储函数对比

* 存储过程与存储函数创建及调用的关键字不同
* 存储过程不一定有返回值，存储函数一定有返回值
* 存储过程一般用于更新，存储函数一般用于查询
* 存储函数可以放在查询语句中（跟系统预定义的函数一样）
* 存储过程可以执行对表和数据的增删改操作

## 其他操作

### 查看

创建信息

```mysql
show create procedure|function getCitiesByProvince;
```

状态信息

```mysql
show procedure|function status [like 'xxxxx%'];
```

更详细的信息

```mysql
select * from information_schema.ROUTINES where xxxxx;
```

### 修改

只能修改相关特性，不能修改存储过程或函数功能

```mysql
alter {procedure|function} 名称 [charateristics];
```

### 删除

```mysql
drop procedure|function 名称;
```

