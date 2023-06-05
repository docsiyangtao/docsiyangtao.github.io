# 服务统一配置

SpringCloud统一配置组件config，实现服务配置文件的统一托管配置~                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     

## 配置中心

### 概述

config（配置）又称为**统一配置中心**。Spring基于Netflix Config进行二次封装而来。顾名思义，就是**将配置统一管理（托管到远程仓库）**，配置统一管理的好处是在日后大规模集群部署服务应用时**使相同的服务配置一致**，日后再修改配置只需要**统一修改**全部同步，**无需一个一个服务手动维护**。

### 图示

![image-20220124204121066](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220124204121066.png)

* **服务端：配置所有的配置文件**
* **客户端：从服务端拉取配置文件进行配置**



## Config

### 前置准备

首先需要准备一个远程git仓库，用于保存配置文件

![image-20220124212625044](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220124212625044.png) 

### Config Server开发

1. 快速构建一个SpringBoot项目，加入服务注册中心，指定端口为6666，服务名为CONFIGSERVER，这一步很重要，等会需要config client指定config server

2. 引入依赖

   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-config-server</artifactId>
   </dependency>
   ```

3. 配置远程仓库的地址

   ```properties
   spring.cloud.config.server.git.uri=https://gitee.com/yangtao8453/springcloud-config
   spring.cloud.config.server.default-label=master
   
   # 私有库需配置用户名和密码
   # spring.cloud.config.server.git.username=
   # spring.cloud.config.server.git.password=
   ```

4. 入口类开启统一配置中心

   ```java
   @SpringBootApplication
   @EnableConfigServer
   public class ConfigServerApplication {...}
   ```
   
5. 将配置文件提交到远程仓库中，依次为：configclient.properties、configclient-dev.properties、configclient-pro.properties

   ```properties
   server.port=2333
   
   spring.application.name=CONFIGCLIENT
   
   spring.cloud.consul.port=8500
   spring.cloud.consul.host=localhost
   
   spring.profiles.active=dev
   ```

   ```properties
   name=dev
   ```

   ```properties
   name=pro
   ```

6. 访问[localhost:7999/configclient-xx.properties](http://localhost:7999/configclient-xx.properties)，获取到的配置信息为

   ![image-20220125234747030](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220125234747030.png)

7. 原因：**访问的configclient-xx.properties文件不存在，因此会拉取默认的configclient.properties**

8. 当访问的配置**文件存在时，会将该文件与configclient.properties合并返回**，如访问：[localhost:7999/configclient-dev.properties](http://localhost:7999/configclient-dev.properties)

   ![image-20220125234925078](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220125234925078.png)

9. 拉取后会在**本地缓存**一份配置文件

   ![image-20220125235211955](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220125235211955.png)

### Config Client

1. 快速构建一个SpringBoot项目，接入服务注册中心，指定端口号为2333

2. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   ```

3. 配置文件，指定config server的地址、注册中心的地址、需要的配置文件

   ```properties
   # 当前configclient统一配置中心在注册中心的服务id
   # 当前configclient统一配置中心在注册中心的服务id
   spring.cloud.config.discovery.service-id=CONFIGSERVER
   
   # 开启config server发现
   spring.cloud.config.discovery.enabled=true
   
   # 配置注册中心
   spring.cloud.consul.host=localhost
   spring.cloud.consul.port=8500
   
   # 获取哪个配置文件：1.分支；2.文件名；3.环境
   spring.cloud.config.label=master
   spring.cloud.config.name=configclient
   spring.cloud.config.profile=dev
   ```

4. 报错：默认从8888拉取配置文件

   ![image-20220126001419784](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220126001419784.png)

   ![image-20220126001050015](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220126001050015.png)

5. 服务以当前的配置文件启动，启动不起来，**应配置更高优先级的配置文件**，**预先从config server拉取配置文件**，**再以application.properties启动**

6. **将config client的配置文件命名为bootstrap.properties**，重新启动，可看到已拉取到配置文件

   ![image-20220126002045851](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220126002045851.png)

7. controller测试

   ```java
   @RestController
   public class TestController {
       @Value("${name}")
       String name;
   
       @GetMapping
       public String test() {
           return "The env is " + name;
       }
   }
   ```

8. 访问[localhost:2333](http://localhost:2333/)

   ![image-20220126002302372](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220126002302372.png)

9. 以后如果想切换环境，直接修改config client的配置文件bootstrap.properties中的**spring.cloud.config.profile**的值

### 手动刷新配置

以上情况中，如果远程仓库的配置文件发生改变，则本地的服务需要重启才能读取配置信息（缓存）。

1. controller类上加入注解 **@RefreshScope**

   ```java
   @RestController
   @RefreshScope
   public class TestController {
   
       @Value("${name}")
       String name;
   
       @GetMapping
       public String test() {
           return "The env is " + name;
       }
   
   }
   ```

2. config client**暴露远端配置端点**（bootstrap.properties中配置）

   ```properties
   management.endpoints.web.exposure.include=*
   ```

3. 修改完远程仓库的配置后，向config client发送一个post方式请求，访问：[localhost:2333/actuator/refresh](http://localhost:2333/actuator/refresh)，返回值会告诉我们哪些信息发生了改变

   ![image-20220126004707646](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220126004707646.png)
   
4. 虽然手动刷新配置不用重启服务，但还是要单独向每个服务发送post请求，服务集群主机一多还是很不方便。除了手动刷新配置，还有自动刷新配置，需配合BUS组件使用，下节再说。



<center>【END】</center>

