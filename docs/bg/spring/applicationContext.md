# ApplicationContext初认识

## 类图

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1684774540324-cc6a4cb0-eb2a-44fd-92f4-7cdb89e2f68f.png)

## BeanFactory

Spring的核心容器，ApplicationContext的父接口（实际上ApplicationContext组合了BeanFactory）

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1684774850495-5596907c-5528-4e14-b431-eff2d44e1490.png)

提供了Bean的相关操作方法，如判断容器中是否包含bean、根据别名或类型获取一个bean，判断bean是否为单例bean等

![image-20230617003731411](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003731411.png)

Spring中创建bean默认是单例的

新建类TestController，并使用@RestController注解将其注册到Spring容器中

```java
@Controller
public class TestController {
}
```

其默认的bean名称为testController（首字母小写），并且这个bean是单例的，可通过如下方式查看

```java
// 获取ApplicationContext对象
ConfigurableApplicationContext context = SpringApplication.run(Demo01Application.class, args);
// 获取单例bean注册表
Class<DefaultSingletonBeanRegistry> clazz = DefaultSingletonBeanRegistry.class;
// 通过反射获取到其中名为singletonObjects的属性
Field singletonObjects = clazz.getDeclaredField("singletonObjects");
singletonObjects.setAccessible(true);
ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
Map<String, Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
// 过滤，查看是否含有名为testController的bean
map.forEach((k, v) -> {
    if (k.contains("testController")) {
        System.out.println(k + " -> " + v);
    }
});
```

## MessageSource

解析消息的接口，常用于国际化，ApplicationContext的父接口之一

![image-20230617003747348](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003747348.png)

在resources目录下，创建资源包messages.properties

```properties
hi=你好
```

messages_en_US.properties

```properties
hi=hello
```

messages_zh_CN.properties

```properties
hi=你好
```

配置中指定基本包名称以及编码格式

```properties
spring.messages.basename=messages
spring.messages.encoding=utf-8
```

由于MessageSource是ApplicationContext的父接口，因此可以使用context的getMessage方法，即可获取解析结果

```java
ConfigurableApplicationContext context = SpringApplication.run(Demo01Application.class, args);
System.out.println(context.getMessage("hi", null, Locale.CHINA));	// 你好
System.out.println(context.getMessage("hi", null, Locale.US));		// hello
```

## ResourcePatternResolver

给定路径，将其解析为资源对象，返回资源数组

![image-20230617003759611](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003759611.png)

获取SpringBoot自动配置的相关文件（META-INFO下spring.factories）

```java
Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");
for (Resource resource : resources) {
    System.out.println(resource);
}
```

![image-20230617003805640](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003805640.png)

## EnvironmentCapable

获取环境或服务配置等信息

![image-20230617003812436](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003812436.png)

如获取JAVA_HOME、服务启动端口等

```java
ConfigurableEnvironment environment = context.getEnvironment();
String javaHome = environment.getProperty("JAVA_HOME");
String port = environment.getProperty("server.port");
```

## ApplicationEventPublisher

事件发布，常用于可能长时间执行或阻塞的异步操作，实现解耦

![image-20230617003820850](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003820850.png)

自定义类继承ApplicationEvent

```java
public class MyApplicationEvent extends ApplicationEvent {
    public MyApplicationEvent(Object source) {
        super(source);
    }
}
```

定义监听组件

```java
@Component
public class MyEventListener {
    @EventListener
    public void handle(MyApplicationEvent event) {
        System.out.println(event.getSource());
    }
}
```

发布事件

```java
@Autowired
private ApplicationEventPublisher eventPublisher;

// ....
eventPublisher.publishEvent(new MyApplicationEvent(123));
```