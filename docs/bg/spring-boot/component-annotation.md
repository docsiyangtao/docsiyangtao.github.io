## @Configuration

此注解用于告知SpringBoot这是一个配置类，@Bean注解可以给容器添加组件，可作用在方法或属性上，可声明组件名称（如不指定则为方法名或驼峰属性名），其返回值 就是组件在容器中的实例。

User类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
}
```

创建组件MyConfig.class

```java
@Configuration(proxyBeanMethods = true)		// 默认为true
public class MyConfig {
    @Bean("tom")	// 注册为bean
    public User user01() {
        return new User(1, "tom");
    }
}
```

在主程序类中测试

```java
// 获取容器
ConfigurableApplicationContext run = SpringApplication.run(SpringBoot01Application.class);

Object tom1 = run.getBean("tom");
Object tom2 = run.getBean("tom");
System.out.println(tom1 == tom2);   // true，由此得知声明的组件默认都是单实例的

MyConfig bean = run.getBean(MyConfig.class);
System.out.println(bean);	//live.yangtao.config.MyConfig$$EnhancerBySpringCGLIB$$f6df835f@1517f633
//获取到的是CGLIB代理的对象，SpringBoot总会检查容器中是否含有该组件，保持组件的单实例

User user1 = bean.user01();
User user2 = bean.user01();
System.out.println(user1 == user2); // true，
```

现修改@Configuration的属性 proxyBeanMethods = false，再测试

```java
MyConfig bean = run.getBean(MyConfig.class);
System.out.println(bean);			// live.yangtao.config.MyConfig@77128dab
User user1 = bean.user01();
User user2 = bean.user01();
System.out.println(user1 == user2); // false，
```

修改后组件每次通过方法获取不同的实例，SpringBoot不会确保组件的单实例

由此引申最佳实践：

1. 配置 类组件之间无依赖关系用Lite模式（proxyBeanMethods = false）加速容器启动过程，减少判断
2. 配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式



## @Import

使用在声明组件的注解上，以数组的方式导入一些类，并将他们声明为组件

```java
@Import({User.class})
@Configuration(proxyBeanMethods = true)
public class MyConfig {
    @Bean("tom")
    public User user01() {
        return new User(1, "tom");
    }
}
```

在主启动类获取User类型的全部组件，将其遍历

```java
String[] beanNamesForType = run.getBeanNamesForType(User.class);
for (String name : beanNamesForType) {
    System.out.println(name);
}
```

打印结果如下：

```
live.yangtao.domain.User
tom
```

可看到导入的类的组件名称默认为全类名。



## @Conditional

条件装配，衍生：容器中有XXBean时注册组件、没有XXBean时注册组件

![image-20220712205858945](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220712205858945.png)

可加在类或方法上，测试如下：

```java
@Import({User.class})
@Configuration(proxyBeanMethods = true)
@ConditionalOnMissingBean(name = "tony")
public class MyConfig {

    @Bean("tom")
    public User user01() {
        return new User(1, "tom");
    }

    @ConditionalOnBean(name = "tom")
    @Bean("tony")
    public User user02() {
        return new User(2, "tony");
    }

    @ConditionalOnMissingClass({"live.yangtao.domain.User"})
    @Bean("pony")
    public User user03() {
        return new User(3, "pony");
    }

}
```

主程序类中测试：

```java
ConfigurableApplicationContext run = SpringApplication.run(SpringBoot01Application.class);

System.out.println(run.containsBean("myConfig"));
System.out.println(run.containsBean("tony"));
System.out.println(run.containsBean("tom"));
System.out.println(run.containsBean("pony"));
```

结果运行结果如下

```
true
true
true
false
```

分析：MyConfig类上使用了`@ConditionalOnMissingBean(name = "tony")`注解，此时容器中并没有`tony`这个bean，因此MyConfig可以顺利被注册为一个组件；加下来代码由上往下执行，注册了bean`tom`，`@ConditionalOnBean(name = "tom")`判断成立，因此注册了bean`tony`；最后注册bean`pony`时，由于MyConfig中引入了User类（live.yangtao.domain.User），因此@ConditionalOnMissingClass不成立，所以没有注册bean`pony`。



## @ImportResource

SpringBoot还允许引入传统Spring项目中用xml配置的bean，如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="luxi" class="live.yangtao.domain.User">
        <property name="id" value="4"/>
        <property name="name" value="luxi"/>
    </bean>

</beans>
```

这是一个User类型的bean，User{id=4, name=zhangsan}，在MyConfig中使用`@ImportResource("classpath:beans.xml")`注解，即可引入创建bean，如下

```java
@Configuration
@ImportResource("classpath:beans.xml")
public class MyConfig {
}
```

主程序类中，测试是否存在名为luxi的bean

```java
System.out.println(run.containsBean("luxi"));
```

结果如下：

```
true
```

