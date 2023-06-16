# 变量与流程控制

## 变量

### 系统变量

1. 系统定义，属于服务器层面
2. MySQL启动以后会生成MySQL服务实例，系统变量定义了当前实例的属性、特征
3. 编译MySQL时参数的默认值，或是配置文件中的参数值
4. 可分为
   * 全局（global）变量，针对所有会话有效，但重启失效
   * 会话（session）变量，默认、重启失效、不同会话不可见
   * 静态变量，MySQL运行期间不能动态修改，属于特殊的全局系统变量
5. 会话中修改了全局变量，则在其他会话中也会生效
6. 系统变量可能是全局的（如max_connections用于限制服务器的最大连接数），可能是全局也可能是会话（如character_set_client用于设置客户端的字符集），还有可能作用域只是当前会话（如pseudo_thread_id用于标记当前会话的MySQL连接id）

<img src="https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230603175637069.png" alt="image-20230603175637069" style="zoom:67%;" />

相关命令

```mysql
-- 查看所有全局变量
show global variables [like '%xxxx%'];

-- 查看所有会话变量
show session variables [like '%xxxx%'];
show variables [like '%xxxx%'];
-- 如果系统变量和会话变量同时含有这个变量，则优先查询会话变量

-- 查看指定的系统变量
select @@global.变量名;

-- 查看指定的会话变量
select @@session.变量名;
select @@变量名;

-- 修改系统变量值
set @@global.变量名 = 变量值;
set global 变量名 = 变量值;
-- 注：如果MySQL重启了，那运行的新MySQL实例会使用配置文件中的变量值

-- 修改会话变量值
set @@session.变量名 = 变量值;
set session 变量名 = 变量值;
```

### 用户变量

1. 用户自定义的变量，以一个 `@` 开头
2. 根据作用范围不同，又分为会话用户变量和局部变量
3. 会话用户变量：作用域与会话变量一样，只在当前session中有效

相关命令

```mysql
-- 定义变量
set @用户变量 = 值;
set @用户变量 := 值;

-- 从表获取
select avg(列名) into @变量名 from 表名;

-- 读取变量
select @用户变量;
```

### 局部变量

1. 只在begin和end语句块中有效，即在存储过程、函数中
2. 使用DECLARE定义一个局部变量，且必须在begin首行中

相关命令

```mysql
-- 声明变量
declare 变量名 类型 [default 默认值];

-- 定义变量
set 变量名 = 值;
set 变量名 := 值;

-- 从表获取
select avg(列名) into @变量名 from 表名;

-- 读取变量
select 变量名;
```

## 定义条件与处理程序

定义条件是事先定义程序执行过程中可能遇到的问题

处理程序定义了在遇到问题时应当财务的处理方式，并且保证存储过程或函数在遇到执行警告或错误时能继续执行

定义条件和处理程序在存储过程、存储函数中都是支持的

### 错误码

`MySQL_ERROR_CODE`：数值类型的错误代码

`SQLSTATE_VALUE`：长度为5的字符串类型错误代码

如插入数据时，非空字段没有默认值

![image-20230607074509144](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230607074509144.png)

其中1364就是错误码，HY000就是错误代码

### 定义条件

定义条件就是给MySQL中的错误码命名，将一个错误名字和指定的错误条件关联，语法如下

```mysql
declare 错误名称 condition for 错误码（或错误名称）
```

如

```mysql
declare field_null_exception condition for 1364;
declare field_null_exception condition for sqlstate 'HY000';
```

### 定义处理程序

为SQL执行过程中发生的某种类型的错误定义特殊的处理程序，语法如下

```mysql
declare 处理方式 handler for 错误类型 处理语句
```

**处理方式：**

* CONTINUE：遇到错误不处理，继续执行
* EXIT：遇到错误马上退出
* UNDO：遇到错误后撤回之前的操作（MySQL暂不支持）

**错误类型：**

* MySQL_ERROR_CODE
* SQLSTATE_VALUE
* 定义的错误名称
* SQLWARNING：匹配所有01开头的SQLSTATE错误代码
* NOT FOUND：匹配所有02开头的SQLSTATE错误代码
* SQLEXCEPTION：匹配所有非SQLWARNING和NOT FOUND的SQLSTATE错误代码

**处理语句：**

* 如果出现上述条件之一，则应采取相对应的触发方式，并执行指定的处理语句，可以是SET xx = xx这样的简单语句，也可以是BEGIN ... END编写的复合语句
