## 概述

![image-20220926222025717](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926222025717.png)

ApplicationContext继承自BeanFactory，实现并增强BeanFactory的功能，BeanFactory才是容器的核心，ApplicationContext【组合】了BeanFactory，如ConfigurableApplicationContext的getBean方法

```java
ConfigurableApplicationContext run = SpringApplication.run(SpringLearningApplication.class, args);
```

```java
public Object getBean(String name) throws BeansException {
   assertBeanFactoryActive();
   return getBeanFactory().getBean(name);
}
```

![image-20220926222748236](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926222748236.png)



## BeanFactory功能

接口方法

![image-20220926222951436](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926222951436.png)

1. getBean获取bean
2. 实现控制反转、依赖注入以及Bean的生命周期中的其他功能



## ApplicationContext接口功能

### 国际化

MessageSource，国际化，语言翻译处理

1. 创建资源包`messages_zh.properties`

   ```properties
   hello=你好
   ```

2. 通过容器的`getMessage`方法获取到翻译后的值

   ```java
   ConfigurableApplicationContext context = SpringApplication.run(SpringLearningApplication.class, args);
   
   String hello = context.getMessage("hello", null, Locale.CHINA);  // Locale.CHINA由浏览器携带至后台
   System.out.println(hello);
   ```

3. 结果如下

   ![image-20220926225222991](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926225222991.png)

### 获取资源

获取系统中的文件或变量

```java
// classpath后加 * 可以获取到jar包中的资源
Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");
for (Resource resource : resources) {
    System.out.println(resource);
}
```

运行结果；

![image-20220926230024482](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926230024482.png)

```java
String javaHome = context.getEnvironment().getProperty("JAVA_HOME");
String serverPort = context.getEnvironment().getProperty("server.port");
System.out.println(javaHome);
System.out.println(serverPort);
```

![image-20220926230223394](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926230223394.png)

### 发布事件

1. 自定义事件

   ```java
   public class MyEvent001 extends ApplicationEvent {
       /**
        * Create a new {@code ApplicationEvent}.
        *
        * @param source the object on which the event initially occurred or with
        *               which the event is associated (never {@code null})
        */
       public MyEvent001(Object source) {
           super(source);
       }
   }
   ```

2. 事件监听器

   ```java
   @Component
   public class MyEvent001Listener {
       @EventListener
       public void handle(MyEvent001 event) {
           System.out.println("MyEvent001Listener处理");
           Object source = event.getSource();
           System.out.println(source);
       }
   }
   ```

3. 发送事件

   ```java
   context.publishEvent(new MyEvent001("hello"));
   ```

4. 运行结果

   ![image-20220926231337465](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220926231337465.png)

