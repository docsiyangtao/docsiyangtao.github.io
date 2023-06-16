# 容器实现

## BeanFactory实现

BeanFactory的原始功能并不丰富，其中很多功能需要后处理器支持

## 建立测试类

```java
@Configuration
static class Config {
    @Bean
    public Bean1 bean1() {
        return new Bean1();
    }

    @Bean
    public Bean2 bean2() {
        return new Bean2();
    }
}

static class Bean1 {
    private static final Logger log = LoggerFactory.getLogger(Bean1.class);

    @Autowired
    private Bean2 bean2;

    public Bean1() {
        log.info("Bean1 init...");
    }

    public Bean2 getBean2() {
        return bean2;
    }
}

static class Bean2 {
    private static final Logger log = LoggerFactory.getLogger(Bean2.class);
    public Bean2() {
        log.info("Bean2 init...");
    }
}
```

## BeanDefinition

创建默认的BeanFactory实现类

```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
```

创建BeanDefinition，指定bean的定义，如class、scope、初始化、销毁

```java
AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class)
                .setScope("singleton")
                .getBeanDefinition();
```

在BeanFactory中注册BeanDefinition

```java
beanFactory.registerBeanDefinition("config", beanDefinition);
```

此时BeanFactory中只有一个BeanDefinition，即`config`

使用AnnotationConfigUtils工具类给BeanFactory添加后处理器，查看BeanDefinition

```java
AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
    System.out.println(beanDefinitionName);
}

Config config = (Config) beanFactory.getBean("config");
config.bean1();
```

运行结果如下

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685033178494-0d001b4c-ede9-422b-a72b-c8aab0969c86.png)

此时BeanFactory中已经有一些其他的BeanDefinition了，从日志中也可以看出此时容器中只创建了config这个bean，但没有创建bean1 bean2。也就说明这些BeanFactory后处理器并没有进行工作！

## BeanFactoryPostProcessor

为了让这些BeanFactory后处理器工作，我们从BeanFactory中获取BeanFactoryPostProcessor类型的bean为BeanFactory创建BeanDefinition

```java
Map<String, BeanFactoryPostProcessor> beanFactoryPostProcessorMap = beanFactory.getBeansOfType(BeanFactoryPostProcessor.class);
beanFactoryPostProcessorMap.values().forEach(processor -> processor.postProcessBeanFactory(beanFactory));
```

再次查看BeanFactory中的BeanDefinition

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685033575606-2cf0a2e7-bb64-4ac7-bd86-1d6c345fac99.png)

可看到bean1和bean2的BeanDefinition也被注册到BeanFactory中了（后处理器internalConfigurationAnnotationProcessor的劳动成果），同时通过config.bean1()方法，可以获取到Bean1的实例

上述BeanFactory后置处理器并不会对Bean进行后置处理，如

```java
System.out.println(beanFactory.getBean(Bean1.class).getBean2());
```

输出结果为null

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685033840512-2d85f758-25eb-42f5-bd8f-72972c1e0819.png)

从以上可知即使Bean2被注入到Bean1类中，也不会创建对应的实例，因为BeanFactory的后置处理器不会对@Autowired声明的Bean进行处理，此时就得使用Bean后处理器，针对Bean生命周期的各个阶段，提供扩展，如org.springframework.context.annotation.internalAutowiredAnnotationProcessor

## BeanPostProcessor

先获取BeanPostProcessor的实例，将其添加到BeanFactory中（注意区分这个步骤和registerAnnotationConfigProcessors、postProcessBeanFactory方法，后两者为注册、为BeanFactory提供功能）

```java
Map<String, BeanPostProcessor> beanPostProcessorMap = beanFactory.getBeansOfType(BeanPostProcessor.class);
beanPostProcessorMap.values().forEach(beanFactory::addBeanPostProcessor);
```

再次运行

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685034421271-a03c47d6-7653-4f11-a543-83ffbfd6ca3c.png)

可看出这次被注入到Bean1中的Bean2已成功实例化

这里有个细节，就是创建实例顺序为 bean1 -> config -> bean2，这是为啥？

增加分隔线，运行如下两段代码

```java
System.out.println("========================");
// beanFactory.preInstantiateSingletons();
System.out.println("========================");
System.out.println(beanFactory.getBean(Bean1.class).getBean2());
```

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685035291782-d89aa48d-1210-4a31-af93-28a40b7f359b.png)

分隔线打印完，有bean被真正调用的时候，才会去实例化，即延迟实例化

## preInstantiateSingletons

BeanFactory中也有个方法，可以立即创建所有单例Bean的实例化对象，如下

```java
System.out.println("========================");
beanFactory.preInstantiateSingletons();
System.out.println("========================");
System.out.println(beanFactory.getBean(Bean1.class).getBean2());
```

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685035473626-c4f31aad-677d-4f4e-ba82-56c660b44a8b.png)

由以上日志可知，在调用了preInstantiateSingletons方法后，会立即进行所有bean的实例化

综上所述，BeanFactory相对于平时的ApplicationContext，有以下特点：

1. 不会主动调用BeanFactory后处理器，为BeanFactory注册BeanDefinition
2. 不会主动添加Bean后处理器
3. 不会主动为单例bean创建实例

## 扩展：@Autowired和@Resource优先级

新增接口以及两个实现类，并在Config中声明两个实现类的bean

```java
static interface Itf {
}

static class Bean3 implements Itf {
    private static final Logger log = LoggerFactory.getLogger(Bean3.class);
    public Bean3() {
        log.info("Bean3 init...");
    }
}

static class Bean4 implements Itf {
    private static final Logger log = LoggerFactory.getLogger(Bean4.class);
    public Bean4() {
        log.info("Bean4 init...");
    }
}

@Configuration
static class Config {
    @Bean
    public Bean1 bean1() {
        return new Bean1();
    }

    @Bean
    public Bean3 bean3() {
        return new Bean3();
    }

    @Bean
    public Bean4 bean4() {
        return new Bean4();
    }

}
```

1. @Autowired是根据类型进行注入，如有不同的实现类，则可使用@Qualifier注入指定的bean或根据变量名注入指定的bean

```java
static class Bean1 {
    private static final Logger log = LoggerFactory.getLogger(Bean1.class);
	@Autowired
    @Qualifier("bean3")
    private Itf itf;

    public Itf getItf() {
        return itf;
    }

}
```

测试

```java
System.out.println(beanFactory.getBean(Bean1.class).getItf());
```

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685552713158-61ccf2ed-6ce5-4372-afba-eb269f56a301.png)

根据日志可知注入的是Bean3

换个写法，这次使用@Qualifier指定bean名称

```java
@Autowired
private Itf bean3;

public Itf getItf() {
    return bean3;
}
```

运行结果如下

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685552877929-7be85437-cdb5-41a5-9809-c856db32b679.png)

注入的同样是Bean3

1. 而@Resource是根据名称进行注入，如果注解中指定了bean名称，则是根据此处指定的bean名称去查找对应bean进行注入

```java
@Resource(name = "bean4")
private Itf bean3;

public Itf getItf() {
    return bean3;
}
```

运行结果如下

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685553106699-9c987eae-e51e-4af2-a37f-876440a07f82.png)

由日志可知注入的是bean4

去掉@Resource中的name值

```java
@Resource
private Itf bean3;
```

再次运行

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685553231759-da3534cb-004e-4ea5-a8ef-b648bd1841da.png)

这次注入的是bean3，验证了没有指定名称时，则根据变量名去注入相应的bean

1. @Autowired和@Resource哪个优先级高？

修改注入如下

```java
@Autowired
@Resource(name = "bean4")
private Itf bean3;
```

运行结果如下

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685553394240-cf8eb760-4bbe-4dea-ab38-9bd80db9c91b.png)

可以看到注入的是bean3，代表这@Autowired的优先级更高，从添加Bean后处理器的步骤入手，查看添加顺序

```java
beanPostProcessorMap.values().forEach(e -> {
    System.out.println("添加Bean后处理器 --> " + e.toString());
    beanFactory.addBeanPostProcessor(e);
});
```

运行结果如下

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685553764255-bcc8aa46-79e6-4395-997b-3364102ffef2.png)

可看到在添加Bean后处理器时，AutowiredAnnotationBeanPostProcessor的添加早于CommonAnnotationBeanPostProcessor的添加，也就意味着Bean先被@Autowired注入了

在为BeanFactory添加Bean后处理器的时候，可以指定添加顺序，如下

```java
beanPostProcessorMap.values()
        .stream()
        .sorted(Objects.requireNonNull(beanFactory.getDependencyComparator()))
        .forEach(e -> {
    System.out.println("添加Bean后处理器 --> " + e.toString());
    beanFactory.addBeanPostProcessor(e);
});
```

再次运行

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685556297030-84ca1acf-0e01-42fe-b910-d3f3fce5cdf5.png)

发现注入的是bean4，排序之后@Resource的优先级更高了

查看DependencyComparator，是在为BeanFactory添加后处理器，即registerAnnotationConfigProcessors中设置的

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685555188291-52a538f1-e99b-4687-9011-d832041a7859.png)

这个比较器重写了OrderComparator的findOrder方法

AutowiredAnnotationBeanPostProcessor中含有一个属性order

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685555905851-afb6ff39-8a1e-450e-8b6f-6b1f9a804146.png)

CommonAnnotationBeanPostProcessor构造方法中，也包含了order值

![img](https://cdn.nlark.com/yuque/0/2023/png/21390724/1685555983002-eaa0c8ae-040c-4578-be3d-e4d9e490fde0.png)

CommonAnnotationBeanPostProcessor的order值更小，在stream排序中，则排在前面，因此@Resource优先级更高。