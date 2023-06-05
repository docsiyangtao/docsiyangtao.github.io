## rpm安装

### 下载

MySQL文件地址：https://downloads.mysql.com/archives/，选择[MySQL Community Server](https://downloads.mysql.com/archives/community/)，选择好版本及安装的系统，将所有需要的文件下载，直接下载**RPM Bundle**即可

### 前置准备

MariaDB检查，有则将其卸载，否则将影响后续MySQL的安装

```bash
rpm -qa | grep mariadb

yum remove mariadb-libs.x86_64
```

### 安装

```bash
tar -xvf mysql-5.7.36-1.el7.x86_64.rpm-bundle.tar
```

安装顺序如下：

![image-20220301091633034](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220301091633034.png)

安装

![image-20220302165737543](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220302165737543.png)

### 密码设置

启动服务，通过日志查看初始密码

```bash
systemctl start mysqld

grep 'temporary password' /var/log/mysqld.log
```

设置密码强度及长度

```mysql
set global validate_password_policy=LOW;

set global validate_password_length=4;
```

修改密码

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```

### 远程设置

支持root用户允许远程连接MySQL数据库

```mysql
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
-- 注：123456值的是密码

flush privileges;
```

创建新的用户

```mysql
grant all on *.* to admin@'%' identified by 'admin' with grant option;
```



## yum安装

### 添加yum源

下载地址：https://dev.mysql.com/downloads/repo/yum/

### 添加yum源

```bash
yum install mysql80-community-release-el7-5.noarch.rpm
```

### 检查源

```bash
yum repolist all | grep mysql

mysql-cluster-7.5-community/x86_64  MySQL Cluster 7.5 Communit disabled
mysql-cluster-7.5-community-source  MySQL Cluster 7.5 Communit disabled
mysql-cluster-7.6-community/x86_64  MySQL Cluster 7.6 Communit disabled
mysql-cluster-7.6-community-source  MySQL Cluster 7.6 Communit disabled
mysql-cluster-8.0-community/x86_64  MySQL Cluster 8.0 Communit disabled
mysql-cluster-8.0-community-source  MySQL Cluster 8.0 Communit disabled
mysql-connectors-community/x86_64   MySQL Connectors Community enabled:      230
mysql-connectors-community-source   MySQL Connectors Community disabled
mysql-tools-community/x86_64        MySQL Tools Community      enabled:      138
mysql-tools-community-source        MySQL Tools Community - So disabled
mysql-tools-preview/x86_64          MySQL Tools Preview        disabled
mysql-tools-preview-source          MySQL Tools Preview - Sour disabled
mysql57-community/x86_64            MySQL 5.7 Community Server disabled
mysql57-community-source            MySQL 5.7 Community Server disabled
mysql80-community/x86_64            MySQL 8.0 Community Server disabled
mysql80-community-source            MySQL 8.0 Community Server disabled
```

### 设置版本

```bash
yum-config-manager --disable mysql57-community
yum-config-manager --enable  mysql80-community
```

### 安装

```bash
yum install mysql-community-server
```

### 查看

```bash
rpm -qa | grep mysql
mysql-community-client-8.0.28-1.el7.x86_64
mysql-community-libs-8.0.28-1.el7.x86_64
mysql80-community-release-el7-5.noarch
mysql-community-client-plugins-8.0.28-1.el7.x86_64
mysql-community-server-8.0.28-1.el7.x86_64
mysql-community-common-8.0.28-1.el7.x86_64
mysql-community-icu-data-files-8.0.28-1.el7.x86_64
```

### 密码设置

8.0 需现修改密码再修改密码策略

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Yangtao123.';
```

密码策略参数变化

```mysql
mysql> show variables like 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 4     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | LOW   |
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
7 rows in set (0.00 sec)
```

其余步骤不再重复
