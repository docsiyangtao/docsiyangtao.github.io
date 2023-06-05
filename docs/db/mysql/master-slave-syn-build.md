# 主从同步的搭建

## MySQL复制原理

1. master将改变记录到二进制日志(binary log)。这些记录过程叫做二进制日志事件，binary log events
2. slave将master的binary log events拷贝到它的中继日志(relay log)
3. slave重做中继日志中的事件，将改变应用到自己的数据库中。MySQL复制是异步的且串行化的

![image-20220731172256291](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220731172256291.png)



## 原则

* 每个slave只能有一个master
* 每个slave只能有一个唯一的服务器ID
* 每个master可以有多个salve
* slave和master的MySQL版本最好一致



## 实操

### 准备

* CentOS 7.6 × 2
* MySQL 5.7
* 关闭防火墙

### Master配置

1. 启用二进制日志记录
2. 设置唯一的服务器ID
3. 重启服务

```cnf
[mysqld]
log-bin=mysql-bin
server-id=1
```

4. 为从库创建用户，方便权限的管理

```mysql
mysql> create user 'slave1'@'%' identified by 'slave1';
Query OK, 0 rows affected (0.00 sec)

mysql> grant replication slave on *.* to 'slave1'@'*';
Query OK, 0 rows affected (0.00 sec)
```

### Slave配置

1. 设置唯一的服务器ID
2. 重启服务

```cnf
[mysqld]
server-id=2
read_only=1
```

### 配置主库通信

1. 查看Master状态，记录下binlog File名称和Position

```mysql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      623 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

2. 从库设置参数

```mysql
mysql> change master to
    -> master_host = '192.168.8.130',
    -> master_user = 'slave1',
    -> master_password = 'slave1',
    -> master_log_file = 'mysql-bin.000001',
    -> master_log_pos = 623;
Query OK, 0 rows affected, 2 warnings (0.01 sec)
```

### 从库启动复制线程

```mysql
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.8.130
                  Master_User: slave1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 623
               Relay_Log_File: localhost-relay-bin.000003
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 623
              Relay_Log_Space: 531
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: e9017ab7-6973-11ec-9f36-000c295f1e3b
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

关注4个参数：

* Slave_IO_State: Waiting for master to send event（从库状态）
* Slave_IO_Running: Yes （从库IO状态）
* Slave_SQL_Running: Yes （从库SQL状态）
* Seconds_Behind_Master: 0 （是否同步）

注：两个SQL的server-uuid不能相同！名称为 'auto.cnf'，位置为 /var/lib/mysql/auto.cnf

### 检验

1. master建表并新增数据

```mysql
mysql> use db1;
Database changed

mysql> create table `tb1`(
    -> `id` int(11) primary key auto_increment,
    -> `name` varchar(255) not null
    -> )engine=innodb default charset=utf8;
Query OK, 0 rows affected (0.01 sec)

mysql> insert into `tb1` (`id`,`name`) values (10001,'zs');
Query OK, 1 row affected (0.01 sec)
```

2. slave中可以看到同步的数据

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use db1;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| tb1           |
+---------------+
1 row in set (0.00 sec)

mysql> select * from `tb1`;
+-------+------+
| id    | name |
+-------+------+
| 10001 | zs   |
+-------+-----+
1 row in set (0.00 sec)
```

3. 完成



## 常用命令

```mysql
show master status;			# 查看主库状态
show slave hosts;			# 查看从库列表
show processlist;			# 查看主库复制线程
show slave status;			# 查看从库状态
start slave;				# 启动从库复制线程
stop slave;					# 停止从库复制
show binary logs;			# 获取binlog文件列表
show binlog events [in 'mysql-bin.000001'];			# 查看第一个或指定的binlog的内容
```



## 复制线程

### Master

```mysql
mysql> show processlist\G
*************************** 1. row ***************************
     Id: 4
   User: slave1
   Host: 192.168.8.128:57298
     db: NULL
Command: Binlog Dump
   Time: 33286
  State: Master has sent all binlog to slave; waiting for more updates
   Info: NULL
*************************** 2. row ***************************
     Id: 5
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: starting
   Info: show processlist
2 rows in set (0.00 sec)
```

* Id：Binlog Dump服务连接从库的复制线程
* User：从库所使用的用户
* Host：从库的地址及端口
* State：主库的当前状态

### Slave

```mysql
mysql> show processlist\G
*************************** 1. row ***************************
     Id: 1
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 33570
  State: Waiting for master to send event
   Info: NULL
*************************** 2. row ***************************
     Id: 2
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 32966
  State: Slave has read all relay log; waiting for more updates
   Info: NULL
*************************** 3. row ***************************
     Id: 5
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: starting
   Info: show processlist
3 rows in set (0.00 sec)
```

* 1是与主库的通信线程
* 2是正在处理日志中更新的SQL线程

