## Tomcat自动配置

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-tomcat</artifactId>
  <version>2.5.0</version>
</dependency>
```



## SpringMVC整合

SpringBoot帮我们整合了DispatcherServlet、视图解析器ViewResolver等SSM项目中需要手动引入的组件，如需查看，可通过主程序类查看

```java
@SpringBootApplication
public class SpringBoot01Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(SpringBoot01Application.class);
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
    }
}
```

![image-20220711231841108](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711231841108.png)



## 包扫描自动配置

默认的包扫描规则，主程序所在目录及其目录下的所有子包的组件，都会被扫描，省去了SSM中扫描包的配置

将包创建在如下位置，不与主程序同目录或在其子目录下

![image-20220711232336086](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711232336086.png)

启动，搜索注册的组件 `testController`，没有结果

![image-20220711232441932](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711232441932.png)

SpringBoot可在主程序类中指定包扫描，扫描指定目录

```java
@SpringBootApplication(scanBasePackages = "live")

等效于
@ComponentScan("live")
```

再次启动，可看到testController已被扫描到

![image-20220711232846976](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711232846976.png)



## 配置映射类

配置文件application.properties中的所有配置都会 映射到一个配置类中，如

```properties
server.prot=8888
```

映射到类ServerProperties类中的port属性

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
public class ServerProperties {
    private Integer port;
    private InetAddress address;
    ...
}
```

注意，自动配置是按需加载的，并不是所有的类都会加载，只有在POM文件中引入了对应的场景启动器才会加载对应的类！

