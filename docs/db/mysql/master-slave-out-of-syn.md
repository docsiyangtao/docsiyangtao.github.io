前两天开发过程中遇到了读写不一致的情况，无奈只能叫公司大佬帮忙解决，事后复个盘，希望以后能自己解决这类问题（假装自己有操作数据库的权限...）                                                                                                                                                                                                                                                                                                 



## 公司的DB结构

![image-20220405165409103](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165409103.png) 



## 环境准备

1. 写库：192.168.8.128；读库：192.168.8.130

2. 写库创建两个库，db1和db2；读库创建一个库，all_db

3. 配置映射，启用主从复制

   ```mysql
   mysql> stop slave;
   Query OK, 0 rows affected (0.00 sec)
   
   mysql> change replication filter replicate_rewrite_db = ((db1,all_db),(db2,all_db));
   Query OK, 0 rows affected (0.00 sec)
   
   mysql> start slave;
   Query OK, 0 rows affected (0.00 sec)
   ```

4. 写库分别在db1和db2中分别创建tb1和tb2，根据读写分离原则，读库all_db中会创建这两个表

   ```mysql
   mysql> use db1;
   Database changed
   mysql> CREATE TABLE IF NOT EXISTS `tb1`(
       -> `ID` INT(8) AUTO_INCREMENT,
       -> `COL1` VARCHAR(20) NOT NULL,
       -> PRIMARY KEY (`ID`)
       -> )ENGINE = InnoDB DEFAULT CHARSET=utf8;
   Query OK, 0 rows affected (0.01 sec)
   
   mysql> use db2;
   Database changed
   mysql> CREATE TABLE IF NOT EXISTS `tb2`(
       -> `ID` INT(8) AUTO_INCREMENT,
       -> `COL1` VARCHAR(20) NOT NULL,
       -> PRIMARY KEY (`ID`)
       -> )ENGINE = InnoDB DEFAULT CHARSET=utf8;
   Query OK, 0 rows affected (0.01 sec)
   ```

   ```mysql
   mysql> use all_db;
   Database changed
   
   mysql> show tables;
   +------------------+
   | Tables_in_all_db |
   +------------------+
   | tb1              |
   | tb2              |
   +------------------+
   2 rows in set (0.00 sec)
   ```



## 问题复现

写库操作带schema导致读写不同步

```mysql
mysql> alter table db1.tb1 add `COL2` VARCHAR(20) NOT NULL;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table db1.tb1 add `COL3` VARCHAR(20) NOT NULL;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

从库查看状态

![image-20220405165415206](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165415206.png)



## 问题解决——方案1

1. 读库先停止读写同步

   ```mysql
   mysql> stop slave;
   Query OK, 0 rows affected (0.00 sec)
   ```

2. 记录错误的SQL语句

3. 设置跳过SQL事件并恢复读写同步

   ```mysql
   mysql> set global sql_slave_skip_counter=1;
   Query OK, 0 rows affected (0.00 sec)
   
   mysql> start slave;
   Query OK, 0 rows affected (0.00 sec)
   ```

4. 继续查看从库状态，是否还有错误的SQL，有则循环1 2 3步，无则进入下一步

5. 修改刚才记录的SQL语句并执行（这里情景为去掉Schema即可，实际中可能为主键重复，删除数据不存在等，需具体情况具体分析）

   ```mysql
   mysql> use all_db;
   Database changed
   mysql> alter table tb1 add `COL2` VARCHAR(20) NOT NULL;
   Query OK, 0 rows affected (0.02 sec)
   Records: 0  Duplicates: 0  Warnings: 0
   
   mysql> alter table tb1 add `COL3` VARCHAR(20) NOT NULL;
   Query OK, 0 rows affected (0.01 sec)
   Records: 0  Duplicates: 0  Warnings: 0
   ```

6. 验证

   ```mysql
   mysql> show create table `tb1`\G
   *************************** 1. row ***************************
          Table: tb1
   Create Table: CREATE TABLE `tb1` (
     `ID` int(8) NOT NULL AUTO_INCREMENT,
     `COL1` varchar(20) NOT NULL,
     `COL2` varchar(20) NOT NULL,
     `COL3` varchar(20) NOT NULL,
     PRIMARY KEY (`ID`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8
   1 row in set (0.00 sec)
   ```



## 问题解决——方案2

*方案1适合及时发现主从不同步且读库和写库相差较小的情况，如相差较大，且数据库很小则推荐方案2*

1. 锁住写库，使用mysqldump备份写库的数据
2. 将备注的数据传给读库
3. 读库停止同步，导入写库备份的数据
4. 读库再重新设置主从同步，注意此时写库的File和Position都已改变，需使用最新值
5. 解锁写库，很重要！



## 拓展

写库执行了读库不能执行的SQL语句是比较常见的导致读写不一致的情况，实际中还有许多原因会导致读写不同步，如：

1. 网络延迟，这个也比较常见，特别是读写库不在同一个局域网时容易出现
2. 负载过高，任意一台主机的SQL线程或IO线程阻塞都会导致读写不同步
3. 读库max_allowed_packet设置过小，无法执行写库传过来的SQL语句
4. 宕机导致bin_log或relaylog文件损坏，应对：设置sync_binlog=1或innodb_flush_log_at_trx_commit=1
5. MySQL自身缺陷或读库版本低于写库，无法执行读库传过来的SQL语句
