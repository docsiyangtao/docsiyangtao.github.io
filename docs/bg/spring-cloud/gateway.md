# 服务网关

SpringCloud的网关服务gateway~                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      

## 网关

### 概述

1. 网关**统一服务入口**，可方便实现对平台众多服务接口进行监控
2. 对访问服务的**身份认证**、防报文重放与防数据篡改
3. 功能调用的**服务鉴权**
4. 响应数据的**脱敏**
5. **流量与并发控制**、基于API调用的计量或计费等。

![image-20220121203310614](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220121203310614.png)



## gateway

### 概述

> This project provides a library for building an API Gateway on top of Spring MVC. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.

* **top of Spring MVC：Spring MVC之上**
* **route to APIs：路由API**
* **provide cross cutting concerns：提供横切关注点** 

### 实践

1. 创建一个springboot项目，端口为7888，注册到服务注册中心

2. 引入依赖，注意：网关服务由于在SpringMVC之上，因此**不能引入springboot的web依赖**（会冲突）

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-gateway</artifactId>
   </dependency>
   ```

3. 在配置文件中配置网关，每条路由需要配**id**（唯一标识）、**uri**（映射到哪个地址）、**predicates**（映射哪些路径，可多个，可写通配符）

   ```yaml
   spring:
     cloud:
       gateway:
         routes:
           - id: openfeignCliento 			# 路由对象唯一标识
             uri: http://localhost:8001/   # 服务路由地址
             predicates:   				# 断言
               - Path=/test,/demo/**
   
           - id: openfeignClientt
             uri: http://localhost:8002/
             predicates:
               - Path=/client2/**
   ```

4. java代码配置，模板如下

   ```java
   @Configuration
   public class GatewayConfig {
       @Bean
       public RouteLocator myLocator(RouteLocatorBuilder builder) {
           return builder.routes()
                   .route("openfeign", r -> r.path("/test")
                   .uri("http://localhost:8001"))
                   .build();
       }
   }
   ```

5. 配置文件配置 & 配置类配置二选一即可，**配置类优先级高于配置文件配置**

6. 映射规则：配置的Path为 /test，地址替换，则网关服务下的 /test 会被映射到 8001 服务下的/test，即localhost:7888/test  -->  localhost:8001/test，如设置通配符，则所有路径都会进行映射

### 负载均衡

网关服务也可以实现负载均衡，gateway也**集成了ribbon**，可以通过注册名去服务注册中心拉取服务IP，再根据负载均衡策略，选择一台实例进行请求转发

配置：**将uri中的IP地址替换为服务名，同时前缀改为【lb】（loadbalance）**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: openfeignCliento 			# 路由对象唯一标识
          uri: lb://OPENFEIGN1/   		# 服务路由地址
          predicates:   				# 断言
            - Path=/test,/demo/**
          filter:						# 过滤器
            - AddRequestHeader=X-Request-red, blue
```

或

```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator myLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("openfeign", r -> r.path("/test")
                .uri("lb://OPENFEIGN1"))
                .build();
    }
}
```

### 断言和过滤

gateway的工作过程

![image-20220123225812486](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220123225812486.png) 

**网关（gateway） = 断言（predicates） + 过滤（filters）**

#### 断言

**当请求到达网关时，网关前置处理，满足断言时放行请求，不满足立即返回**。

内置断言：

- After=2020-07-21T11:33:33.993+08:00[Asia/Shanghai]：指定日期之后的请求进行路由
- Before=2020-07-21T11:33:33.993+08:00[Asia/Shanghai]：指定日期之前的请求进行路由
- Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]：指定日期之间的请求进行路由
- Cookie=username, yang：基于指定cookie的请求进行路由
- Cookie=username,[A-Za-z0-9]+： 基于指定cookie的请求进行路由，不关心值（可正则）
- Header=X-Request-Id, \d+：基于请求头中的指定属性的正则匹配路由（这里全是整数）
- Method=GET,POST：基于指定的请求方式请求进行路由

![image-20220123231126451](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220123231126451.png) 

官方更多: https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.3.RELEASE/reference/html/#the-cookie-route-predicate-factory

#### 过滤

**路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由筛选器的作用域是特定路由。springcloudgateway包括许多内置的GatewayFilter工厂。**

内置过滤器

* AddRequestHeader=X-Request-red, blue：给请求加上请求头
* AddRequestParameter=red, blue：增加请求参数的filterr
* AddResponseHeader=X-Response-Red, AAA：增加响应头filter
* PrefixPath=/emp：增加前缀的filter，如访问：/list，经过网关，访问的就是/emp/list
* StripPrefix=2：去掉2个前缀，如访问/emp/vip/list，经过网关后，访问的就是/list

**全局Filter**

```java
@Configuration
public class CustomerGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();
        System.out.println("经过全局Filter处理...");
        Mono<Void> filter = chain.filter(exchange);
        System.out.println("响应回来...");
        return filter;
    }

    //filter的顺序，-1最先执行，之后0、1、2、...
    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 路由规则

配置：暴露所有的路由端口

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"   #开启所有web端点暴露
```

访问：http://localhost:7888/actuator/gateway/routes，可以查看所有的路由规则

```json
[
	{
		"predicate": "Paths: [/test/**], match trailing slash: true",
        "route_id": "openfeignCliento",
        "filters": [ ],
        "uri": "http://localhost:8001/",
        "order": 0
	},
	{
        "predicate": "Paths: [/client2/**], match trailing slash: true",
        "route_id": "openfeignClientt",
        "filters": [ ],
        "uri": "http://localhost:8002/",
        "order": 0
	}
]
```



<center>【END】</center>
