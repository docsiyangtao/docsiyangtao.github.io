## 概述

Remote Dictionary Server，远程词典服务，基于内存的键值型NoSQL的数据库



## 特征

* 键值型，支持多种不同的数据结构
* 单线程，每命令具备原子性
* 低延迟，速度快（基于内存、IO多路复用、良好的编码）
* 支持数据的持久化
* 支持主从集群、分片集群（水平扩展）
* 支持多语言客户端



## 安装

https://www.redis.io/

1. C语言依赖

   ```bash
   yum install -y gcc tcl
   ```

2. 上传压缩包并解压

   ```bash
   tar -zxvf redis.xxxx.tar.gz
   ```

3. 进入解压目录，编译

   ```bash
   make & make install
   ```

4. 默认安装路径`/usr/local/bin`

5. 启动（安装已自动加入环境变量）

   ```bash
   redis server
   ```

6. 自启动

   ```bash
   systemctl enable redis
   ```

   



## 配置文件

* 监听地址修改（特定IP可以访问）

  ```conf
  bind 0.0.0.0
  ```

* 守护进程（后台运行）

  ```conf
  daemonize yes
  ```

* 密码

  ```conf
  requirepass xxxxx
  ```

* 端口

  ```conf
  port 6379
  ```

* 工作目录（默认运行redis-server的地方，日志、持久化文件存放位置）

  ```conf
  dir .
  ```

* 数据库数量

  ```conf
  databases 1
  ```

* 最大内存

  ```conf
  maxmemory 512mb
  ```

* 日志文件（默认为空）

  ```conf
  logfile "xxxx.log"
  ```

  