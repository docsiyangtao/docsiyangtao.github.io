# 服务断路及监控

SpringCloud的服务短路及监控工具——Hystrix，虽然已经停止维护，但还是有必要了解一下，后续再更SpringCloud Alibaba的sentinel                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         

## 服务雪崩

### 概述

微服务在进行服务调用时，**下游服务崩溃导致上级服务积压过多突发的请求**，慢慢地，**线程资源耗尽**，服务崩溃，接着压力**传递到更上一级服务**，最终导致链路中**所有服务都崩溃**，即**服务雪崩**。

### 过程 

<img src="https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220119200302699.png" alt="image-20220119200302699" style="zoom:67%;" />  

1. Service C突然崩溃，导致Service B积压过多的请求，没有释放资源
2. Service B由于资源耗尽，进而崩溃，导致Service A积压过多请求
3. Service A也耗尽资源，崩溃
4. 链路中所有自C开始的上游服务全部崩溃

### 解决

#### 服务熔断

当发现某个服务异常的时候，直接熔断整个服务，给上游服务（调用者）返回一个友好的错误提示，以此防止上游服务线程阻塞、耗尽资源、崩溃、进而引发雪崩效应（这有点类似家里的保险丝，用电功率过大保险丝就熔断，防止发生意外）；等待目标服务情况好转，主机有更多的空闲资源了，则继续恢复调用。

![image-20220119201204390](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220119201204390.png)

#### 服务降级

当服务压力剧增的时候根据当前的业务情况对一些服务和页面有策略的降级，缓解服务器的压力，保证核心任务能够完成（如双十一下单高峰期，只保证能够完成下单，但是下单成功后跳转到指定页面这种边缘服务则无所谓，可直接返回一个友好的报错页面，让用户自行刷新查看）。

![image-20220119203031308](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220119203031308.png)

#### 总结

1. 服务熔断和服务降级**都是为了系统的可行性考虑**
2. 熔断是出于**服务故障**而触发，而降级是出于系统的**整体负荷**和核心业务而触发
3. 熔断必定触发降级，所以**熔断也是降级的一种**
4. 熔断是“**处理**”，降级是“**预防**”



## Hystrix

### 概述

> In a distributed environment, inevitably some of the many service dependencies will fail. Hystrix is a library that helps you control the interactions between these distributed services by adding latency tolerance and fault tolerance logic. Hystrix does this by isolating points of access between the services, stopping cascading failures across them, and providing fallback options, all of which improve your system’s overall resiliency.

* **adding latency tolerance and fault tolerance logic：添加延迟容忍和容错逻辑**
* **by isolating points of access between the services：分隔服务之间的访问点**
* **stopping cascading failures across them：停止跨服务的级联故障**
* **providing fallback options：提供回退选项**

### 实践——下游

1. 快速创建SpringBoot项目，端口号10086，加入consul注册中心

2. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

3. 入口类开启熔断功能

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   @EnableCircuitBreaker
   public class HystrixApplication {...}
   ```

4. 默认10s超过20个请求，或10s超过50%的请求失败则会将服务断开；现定义失败后的快速响应调用的方法（返回值和参数类型需对应），用抛出异常模拟服务调用失败

   ```java
   @RestController
   public class TestController {
   
       @GetMapping
       @HystrixCommand(fallbackMethod = "testFallBack")
       public String test(Integer integer) {
           if (integer < 0) {
               throw new RuntimeException("模拟出错");
           }
           return "hello";
       }
   
       public String testFallBack(Integer integer) {
           return "服务器繁忙，请稍后再试！";
       }
   
   }
   ```

5. 10s内连续请求失败后，断路器打开，服务熔断，对于正确的请求也会返回快速响应

   ![image-20220119213243728](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220119213243728.png)

6. 断路器打开条件：

   * 10s内超20个请求次数
   * 10s内超50%的请求失败
   * 断路器打开5s后，进入半开状态，如果一个请求成功则关闭断路器，否则继续打开断路器

   ![image-20220119213945367](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220119213945367.png)

7. 如果不想在@HystrixCommand中自定义处理方法，则可以指定默认的快速响应方法

   ```java
   @HystrixCommand(fallbackMethod = "testFallBack", defaultFallback = "defaultFallBack")
   public String test() {...}
   
   //默认的快速响应方法，返回类型为String，空参
   public String defaultFallBack() {
       return "服务器繁忙，请稍后再试！";
   }
   ```

### 实践——OpenFeign & Hystrix

1. 再新建一个SpringBoot项目，加入服务注册中心，端口为10087，引入OpenFeign（默认含Hystrix）

2. 配置文件开启hystrix支持

   ```properties
   feign.hystrix.enabled=true
   ```

3. 被调用的服务不做处理

4. 调用方Feign接口，@FeignClient注解指定fallback的值，为一个类

   ```java
   @FeignClient(value = "HYSTRIX", fallback = HystrixClientFallBack.class)
   public interface HystrixClient {
   
       @GetMapping("/test")
       String test(@RequestParam("integer") Integer integer);
   
   }
   ```

5. @FeignClient中fallback指定的类为Feign接口的实现类，重写的方法就是快速响应的方法

   ```java
   @Component
   public class HystrixClientFallBack implements HystrixClient {
       @Override
       public String test(Integer integer) {
           return "服务器繁忙，请稍后再试！";
       }
   }
   ```

6. 调用方controller

   ```java
   @RestController
   public class TestController {
   
       @Qualifier("com.yangtao.feign.HystrixClient")
       @Autowired
       HystrixClient hystrixClient;
   
       @GetMapping
       public String test(@RequestParam("integer") Integer integer) {
           String test = hystrixClient.test(integer);
           return test;
       }
   
   }
   ```

7. 当满足断路器开启条件时，即使被调用方不开启断路器支持，调用方也会开启断路器

   ![image-20220119221720495](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220119221720495.png)



## Hystrix Dashboard

### 概述

Hystrix仪表盘，收集了关于**每个HystrixCommand的度量**，以图形化界面显示每个断路器的运行状况。

使用时需要创建单独的springboot项目，该服务仅仅只有监控功能，因此没有必要加入注册中心。

### 实践

1. 创建springboot项目，端口为9888

2. 引入hystrix dashboard依赖

   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
   </dependency>
   ```

3. 入口类开启dashboard

   ```java
   @SpringBootApplication
   @EnableHystrixDashboard
   public class HystrixDashBoardApplication {...}
   ```

4. 启动项目，访问图形化界面，http://localhost/hystrix

   ![image-20220120082139565](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120082139565.png)

5. 坑1：被监控的项目中入口类或新建配置类，加入监控路径配置（否则找不到监控的服务），并启动监控项目

   ```java
   @Configuration
   public class HystrixDashboardConfig {
       @Bean
       public ServletRegistrationBean getServlet() {
           HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
           ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
           registrationBean.setLoadOnStartup(1);
           registrationBean.addUrlMappings("/hystrix.stream");
           registrationBean.setName("HystrixMetricsStreamServlet");
           return registrationBean;
       }
   }
   ```

6. 坑2：新版本中springcloud将jquery版本升级为3.4.1，所以需要修改monitor.ftlh文件，如下：

   ```javascript
   $(window).load(function()  修改为   $(window).on("load",function()
   ```

7. \repository\org\springframework\cloud\spring-cloud-netflix-hystrix-dashboard\2.2.3.RELEASE

   ![image-20220120085034155](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120085034155.png)

8. 监控路径去掉/actuator，如：http://localhost:10087/hystrix.stream

9. 调用服务，查看失败率，断路器是否打开

   ![image-20220121005638286](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220121005638286.png)

### 展望

目前Hystrix DashBoard已经停止维护，所以存在较多未解决的bug，现在更推荐使用SpringCloud Alibaba的**sentinel**（后续更新~）。



<center>【END】</center>
