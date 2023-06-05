# 微服务之SpringCloud

SpringCloud基础学习阶段的笔记，持续更~

## 微服务概述

官网: https://www.martinfowler.com/articles/microservices.html

> In short, the microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.

重点词：

* a suite of small services：一系列微小的服务
* running in its own process：独立的进程
* often an HTTP resource API：通常使用HTTP通信（暴露API）
* built around business capabilities：基于业务拆分成单元
* independently deployable by fully automated deployment machinery：独立、自动部署
* bare minimum of centralized management of these services：最低限度的集中管理（分布式）
* different programming languages and use different data storage technologies：不同的语言、数据存储技术

​        **微服务是一种架构，这种架构是将单个的整体应用程序分割成更小的项目关联的独立的服务。一个服务通常实现一组独立的特性或功能，包含自己的业务逻辑和适配器。各个微服务之间的关联通过暴露api来实现。这些独立的微服务不需要部署在同一个虚拟机，同一个系统和同一个应用服务器中。**



## 集群和分布式

**集群：**同一种软件服务的多个服务节点共同为系统提供服务过程，如MySQL集群，称之为服务集群。

**分布式：**不同软件集群（MySQL集群、Redis集群）共同为一个系统提供服务，这个系统称之为分布式集群。

个人理解：一个项目组就好比一个分布式系统，里面有PM、需求、一群开发、一群开发（好比集群）



## 架构对比

### 单体应用

![image-20220405165835000](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165835000.png)

#### 优点

* 项目初期开发、测试、部署都很方便

#### 缺点

- 后期项目越来越大维护困难
- 想采用新的架构或语言只能重新来过
- 系统可靠性低，一个漏洞影响整个应用

### 微服务架构应用

![image-20220405165838825](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165838825.png)

#### 优点

- 每个服务**高度自治**，一个服务拆分为多个职责单一的服务
- 服务之间**松耦合**，即使一个服务出错，不影响整体服务的运行
- 每个服务可用**不同的语言或技术**，只需暴露HTTP接口

#### 缺点

- 复杂性提高
- 集群部署，运维难度及压力提高
- 服务治理和监控较难，如负载均衡、服务雪崩、配置管理



## 架构演变

![image-20220405165843367](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165843367.png)

* **All in One  单一架构**

  所有功能都写在一个应用，减少成本，此时期ORM是关键（解决对象关系映射）

* **Vertical Application  垂直架构**

  三层架构，涌现很多加速开发的框架，此时期MVC是关键

* **Distributed Service  分布式服务架构**

  抽取核心业务为独立服务，此时期提高业务复用及整合的分布式框架（RPC）是关键

* **Elastic Computing  微服务架构**

  服务压力不同，需提高集群利用率，此时期提高机器利用率的资源调度和治理中心（SOA）是关键



## 解决方案

### Dubbo

阿里的分布式治理框架，2011年初出茅庐，刚出时很火，不到一年随着负责人的离职而停止维护，直至17年9月才又开始升级及维护，目前不是微服务的最佳解决方案。

**官网：**https://dubbo.apache.org/zh/

![image-20220405165847783](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165847783.png)



### SpringCloud

### 概述

**细分：**

* SpringCloud Netflix
* SpringCloud alibaba
* SpringCloud Spring

**官网：**https://spring.io/projects/spring-cloud

> Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer’s own laptop, bare metal data centres, and managed platforms such as Cloud Foundry.

**通俗理解：**SpringCloud是一个工具集，提供了配置管理、服务发现、断路器、智能路由等等工具，可以帮助我们快建立这些模式的服务和应用程序。

### 核心架构及组件

![image-20220405165851703](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165851703.png)

- 注册中心：eureka、consul、nacos
- 负载均衡：ribbon
- 服务调用：openfeign
- 服务断路：hystrix
- 服务监控：hystrix dashboard
- 网关组件：zuul、gateway
- 配置中心：config
- 消息总线：bus

### 版本

由于SpringCloud各个组件的开发进度可能不同，因此不能简单的使用过版本号来命名版本，所以才用了英文单词命名，从第一个版本到现在依次为：Angel、Brixton、Camden、Dalston、Edgware、Finchley、Greenwich、Hoxton，Finchley版本之前的都是基于SpringBoot 1.x，Finchley开始基于SpringBoot 2.x，详情可在官网查看。

### 基础环境搭建

```xml
<!-- 继承SpringBoot父项目 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
</parent>

<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
    <spring.cloud-version>Hoxton.SR6</spring.cloud-version>
</properties>

<dependencyManagement>
    <dependencies>
        <!-- 维护SpringCloud版本 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring.cloud-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

