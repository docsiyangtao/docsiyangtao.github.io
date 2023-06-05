# 服务注册中心

SpringCloud学习笔记——服务注册中心，讲了Eureka和Consul，Zookeeper和Nacos后续再更~                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  

## 概述

在整个微服务架构单独抽取一个服务，该服务没有任何业务功能，仅用来完成整个微服务系统的**服务注册**、**服务发现**、**服务健康状态的监控和管理**、**以及服务元数据信息存储**。

![image-20220405170305807](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405170305807.png)

**具体作用：**

* 可以对所有的微服务的**信息进行存储**，如微服务的名称、IP、端口等。
* 可以在进行服务调用时，通过服务发现**查询可用的微服务列表及网络地址**进行服务调用。
* 可以对所有的微服务进行心跳检测，如发现某实例长时间无法访问，就会从服务注册表移除该实例。



## Eureka

### 概述

Eureka是Netflix开发的服务发现框架，本身是一个**基于REST的服务**。SpringCloud将它集成在其子项目spring-cloud-netflix中，以**实现SpringCloud的服务注册和发现功能**。eureka2.0版本目前已停止更新，1.0还在稳定使用。

包含两个组件：**`Eureka Server`**、**`Eureka Client`**

### Eureka Server实践

#### 创建

1. 创建SpringBoot项目

2. 引入Eureka的Maven依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ```

3. 编写配置文件

   ```properties
   server.port=8761
   spring.application.name=eureka
   ```
   
4. 入口类加上开启EurekaServer的注解

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class EurekaServerApplication {...}
   ```

#### 验证

访问 http://localhost:8761/

![image-20220828220911823](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828220911823.png)

可正常访问服务注册中心，服务列表中有EUREKA-SERVER

#### 报错

**com.netflix.discovery.DiscoveryClient    : DiscoveryClient_EUREKA-SERVER/localhost:eureka-server:8761 - was unable to refresh its cache! status = Cannot execute request on any known server**

![image-20220828220602345](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828220602345.png)

**原因：**eureka包含两个组件，包含eureka client，项目启动时会将自己作为客户端立即进行注册，注册时server还没准备完成，所以出现这个错误。

**解决：**

1. 关闭eureka client的立即注册
2. 让当前应用仅作为服务注册中心，不做客户端

```properties
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
```

再次验证

![image-20220828221201444](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828221201444.png)

服务启动没有报异常，且注册中心页面已经找不到EUREKA-SERVER了

### Eureka Client实践

#### 创建

注：这里的eureka client模拟的是基于业务拆分出来的一个个微服务，实际开发并没有这种服务

1. 创建SpringBoot项目

2. 引入Eureka的Maven依赖

   ```java
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

3. 编写配置文件

   ```properties
   server.port=10001
   spring.application.name=eureka-client
   server.servlet.context-path=/eureka-client
   
   # 服务注册中心的地址
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka
   ```

4. 入口类加上开启EurekaClient的注解

   ```java
   @SpringBootApplication
   @EnableEurekaClient
   public class EurekaClientApplication {...}
   ```

#### 验证

![image-20220828221411549](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828221411549.png)

EUREKA-CLIENT已经注册到服务注册中心了 

### Eureka的自我保护机制

**`EMERGENCY!  EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT.  RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.`**

> Eureka servers will enter self preservation mode if they detect that a larger than expected number of registered clients have terminated their connections in an ungraceful way, and are pending eviction at the same time. This is done to ensure catastrophic network events do not wipe out eureka registry data, and having this be propagated downstream to all clients.
>
> To better understand self preservation, it help to first understand how does eureka clients 'end' their registration lifecycle. The eureka protocol requires clients to execute an explicit unregister action when they are permanently going away. For example, in the provided java client, this is done in the shutdown() method. any clients that fails 3 consecutive heartbeat renewals is considered to have an unclean termination, and will be evicted by the background eviction process. It is when > 15% of the current registry is in this later state, that self preservation will be enabled.
>
> When in self preservation mode, eureka servers will stop eviction of all instances until either:
>
> 1. the number of heartbeat renewals it sees is back above the expected threshold, or
> 2. self preservation is disabled (see below)
>
> Self preservation is enabled by default, and the default threshold for enabling self preservation is > 15% of the current registry size.

1. 自我保护机制**默认开启**

2. 超15%的服务出现了连续3次的心跳更新失败（每30s发送一次），那么Eureka服务就会进入自我保护状态

3. eureka并不会因为没有收到某一次心跳而清除注册表

4. 自我保护模式下，停止清除注册表中的所有实例，**宁可保留错误的注册信息，也不盲目注销可能健康的实例**

5. 关闭自我保护机制：

   server设置，且设置3000ms没有收到心跳就剔除该服务

   ```properties
   eureka.server.enable-self-preservation=false
   eureka.server.eviction-interval-timer-in-ms=3000
   ```

   关闭后会提示：**`THE SELF PRESERVATION MODE IS TURNED OFF. THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.`**

   client设置，向server发送心跳的时间设置为1s（默认30），server收到最后一次心跳后等待时间上限设置为2s（默认90），即告诉server，2s没有收到我的心跳就把我剔除

   ```properties
   eureka.instance.lease-renewal-interval-in-seconds=1
   eureka.instance.lease-expiration-duration-in-seconds=2
   ```

### Eureka Server集群搭建

拓扑图

![image-20220405170324691](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405170324691.png)

1. 复制3个Server项目，通过运行参数修改启动端口

   ![image-20220405170329353](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405170329353.png)

2. 对每个复制的项目都修改配置文件（这里只写出节点1的配置）：节点1向节点2、3注册；节点2向节点1、3注册；节点3向节点1、2注册

   ```properties
   eureka.client.service-url.defaultZone=http://localhost:9002/eureka,http://localhost:9003/eureka
   ```

3. client配置文件中，服务注册中心的地址填上3个eureka的地址

   ```properties
   eureka.client.service-url.defaultZone=http://localhost:9001/eureka,http://localhost:9002/eureka,http://localhost:9003/eureka
   ```

4. 启动，client会随机挑选一个服务注册中心

5. 当被client选中的服务注册中心宕机，若干秒以后，client会自动从另外两个服务注册中心中挑选一个作为服务注册中心，即完成了Eureka Server的集群搭建，实现了注册中心的高可用



## consul

### 概述

consul是一个可以提供**服务发现**，**健康检查**，**多数据中心**，Key/Value存储等功能的分布式服务框架，用于实现分布式系统的服务发现与配置。

Consul用Golang实现，因此具有**天然可移植性**(支持Linux、Windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署。

官网：https://www.consul.io/

### Server实践

启动

```bash
consul agent -dev
```

![image-20220828223412353](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828223412353.png)

* dc1为datacenter，默认为dc1，可以指定数据中心启动

  ```bash
  consul agent -dev -datacenter dc2
  ```

* services：当前consul服务中注册的服务列表，默认会自己注册自己，即出现一个consul服务
* nodes：用来查看consul的集群节点

### Client实践

1. 创建一个SpringBoot项目，引入consul-discovery依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-consul-discovery</artifactId>
   </dependency>
   ```

2. 配置文件指定consul-server服务注册中心的地址

   ```properties
   server.port=10002
   spring.application.name=consul-client
   
   # 服务注册中心的地址及端口
   spring.cloud.consul.host=localhost
   spring.cloud.consul.port=8500
   # 默认值为服务名，可更改
   spring.cloud.consul.discovery.service-name=${spring.application.name}
   ```

3. 通用的服务注册中心注解（除Eureka），@EnableDisCoveryClient

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class ConsulClientApplication {...}
   ```

4. 启动服务，查看services

   ![image-20220828224005883](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828224005883.png)

5. 出错，consul-client无法正常响应server的心跳，需要手动引入健康检查的依赖，client才能响应server

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

6. 或者关闭consul客户端的健康检查

   ```properties
   spring.cloud.consul.discovery.register-health-check=false
   ```

7. 查看services

   ![image-20220828224238542](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828224238542.png)



## Eureka VS Consul

### CAP

指的是在一个分布式系统中，**一致性（Consistency）**、**可用性（Availability）**、**分区容错性（Partition tolerance）**。CAP 原则指的是，这三个要素**最多只能同时实现两点**，不可能三者兼顾。

* 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。
* 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。
* 分区容忍性（P），就是高可用性，一个节点崩了，并不影响其它的节点。

### Eureka

Eureka中没有使用任何的数据强一致性算法保证不同集群间的Server的数据一致，仅通过数据拷贝的方式争取注册中心数据的最终一致性，**虽然放弃数据强一致性但是换来了Server的可用性**，降低了注册的代价，**提高了集群运行的健壮性**。

### Consul

基于Raft算法，**Consul提供强一致性的注册中心服务**，但是由于**Leader节点承担了所有的处理工作**，势必加大了注册和发现的代价，**降低了服务的可用性**。通过Gossip协议，Consul可以很好地监控Consul集群的运行，同时可以方便通知各类事件，如Leader选择发生、Server地址变更等

### 对比

|  组件  |  语言  | 一致性算法 | 健康检查 |
| :----: | :----: | :--------: | :------: |
| Eureka |  Java  |     无     |   支持   |
| Consul | Golang |    Raft    |   支持   |



<center>【END】</center>
