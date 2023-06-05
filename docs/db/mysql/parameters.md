# 配置文件

## 安全策略

1. 配置文件启动或停用插件

   ```bash
   [mysqld] 
   plugin-load-add=validate_password.so 		# linux环境下为so，Windows环境下为dll
   # ON/OFF/FORCE/FORCE_PLUS_PERMANENT: 是否使用该插件(及强制/永久强制使用) 
   validate_password=FORCE_PLUS_PERMANENT
   ```

   查看

   ```mysql
   SELECT PLUGIN_NAME, PLUGIN_LIBRARY, PLUGIN_STATUS, LOAD_OPTION
   FROM INFORMATION_SCHEMA.PLUGINS
   WHERE PLUGIN_NAME = 'validate_password';
   ```

2. 运行时安装

   ```mysql
   INSTALL PLUGIN validate_password SONAME 'validate_password.so';
   ```

3. 卸载

   ```mysql
   UNINSTALL PLUGIN validate_password;
   ```

4. 查看

   ```mysql
   show VARIABLES like 'validate_password%';
   ```

   MySQL 8之前的版本

   ```shell
   +--------------------------------------+-------+
   | Variable_name                        | Value |
   +--------------------------------------+-------+
   | validate_password_check_user_name    | OFF   |
   | validate_password_dictionary_file    |       |
   | validate_password_length             | 4     |
   | validate_password_mixed_case_count   | 1     |
   | validate_password_number_count       | 1     |
   | validate_password_policy             | LOW   |
   | validate_password_special_char_count | 1     |
   +--------------------------------------+-------+
   7 rows in set (0.00 sec)
   ```

   MySQL 8开始

   ```shell
   +--------------------------------------+--------+
   | Variable_name                        | Value  |
   +--------------------------------------+--------+
   | validate_password.check_user_name    | ON     |
   | validate_password.dictionary_file    |        |
   | validate_password.length             | 8      |
   | validate_password.mixed_case_count   | 1      |
   | validate_password.number_count       | 1      |
   | validate_password.policy             | MEDIUM |
   | validate_password.special_char_count | 1      |
   +--------------------------------------+--------+
   7 rows in set (0.00 sec)
   ```

5. 设置

   ```mysql
   -- 设置密码长度
   set global validate_password_length=1;
   
   -- 其余参数：密码策略、大小写、特殊字符、数字数量等，可通过【4】的命令查看
   ```



## 字符集

1. 查看（全局/默认）

   ```mysql
   show variables like 'character%'; 
   ```

   MySQL 8之前，服务端latin1不支持中文

   ```shell
   +--------------------------+----------------------------+
   | Variable_name            | Value                      |
   +--------------------------+----------------------------+
   | character_set_client     | utf8                       |
   | character_set_connection | utf8                       |
   | character_set_database   | latin1                     |
   | character_set_filesystem | binary                     |
   | character_set_results    | utf8                       |
   | character_set_server     | latin1                     |
   | character_set_system     | utf8                       |
   | character_sets_dir       | /usr/share/mysql/charsets/ |
   +--------------------------+----------------------------+
   8 rows in set (0.03 sec)
   ```

   MySQL 8开始，默认为utf8mb4，支持更多字符

   ```shell
   +--------------------------+--------------------------------+
   | Variable_name            | Value                          |
   +--------------------------+--------------------------------+
   | character_set_client     | utf8mb4                        |
   | character_set_connection | utf8mb4                        |
   | character_set_database   | utf8mb4                        |
   | character_set_filesystem | binary                         |
   | character_set_results    | utf8mb4                        |
   | character_set_server     | utf8mb4                        |
   | character_set_system     | utf8mb3                        |
   | character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
   +--------------------------+--------------------------------+
   8 rows in set (0.00 sec)
   ```

2. 查看（具体）

   ```mysql
   -- 具体数据库
   show create database 数据库名;
   
   -- 具体表
   show create table 表名;
   
   -- 查看表的比较规则
   show table status like '表名';
   show table status from 数据库名 like '表名';
   ```

3. 修改

   MySQL 5.7或之前，在配置文件中配置，重启后，**新建的**数据库就默认是utf8字符集了

   ```bash
   [mysqld]
   
   character_set_server=utf8
   ```

4. 修改旧数据库的编码，注：只能修改数据库的编码，对于已插入的数据编码不回改变，需重新插入

   ```mysql
   alter database 数据库名 character set '编码方式';
   ```

   

## 大小写

1. 查看

   Linux环境下MySQL对大小写是敏感的，通过查看参数为`lower_case_table_names`，查看命令如下：

   ```mysql
   show variables like '%lower_case_table_names%';
   ```

   ```shell
   +------------------------+-------+
   | Variable_name          | Value |
   +------------------------+-------+
   | lower_case_table_names | 0     |
   +------------------------+-------+
   1 row in set (0.00 sec)
   ```

   值为0表示大小写敏感（数据库名、表名、表的别名、变量名）

2. 设置

   配置文件中加入：

   ```bash
   [mysqld]
   
   lower_case_table_names=1
   ```

   注：重启数据库实例前需要将原来的数据库和表转换为小写！MySQL8下需要删除数据目录（/var/lib/mysql），在配置文件（/etc/my.cnf）中添加以上参数，再重启服务

3. 建议

   平时在书写SQL时

   * 关键字和函数名全部大写
   * 数据库名、表名、表别名、字段名、字段别名全部小写



## 严格/宽松模式

1. 查看

   ```mysql
   select @@session.sql_mode;
   
   select @@global.sql_mode;
   
   show variables like 'sql_mode';
   ```

2. 设置为严格模式

   临时：会话/服务重启后失效

   ```mysql
   set session sql_mode = 'STRICT_TRANS_TABLES';
   
   set global sql_mode = 'STRICT_TRANS_TABLES';
   ```

   永久：在配置文件中添加

   ```bash
   [mysqld]
   
   sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR
   _DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
   ```

3. 建议

   一定要开启严格模式！
