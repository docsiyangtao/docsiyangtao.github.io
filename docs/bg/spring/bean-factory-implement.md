## BeanFactory缺点

### 自定义BeanFactory

1. 需要的类

   ```java
   @Configuration
   class Config {
       @Bean
       public Bean1 bean1() {
           System.out.println("开始构造bean1");
           return new Bean1();
       }
       @Bean
       public Bean2 bean2() {
           System.out.println("开始构造bean2");
           return new Bean2();
       }
   }
   
   class Bean1 {
       @Autowired
       Bean2 bean2;
       public Bean2 getBean2() {
           return bean2;
       }
   }
   
   static class Bean2 {
   }
   ```

2. 使用beanFactory创建一个BeanDefinition

   ```java
   DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
   // bean的定义（class、scope、初始化、销毁）
   AbstractBeanDefinition configBeanDefinition = BeanDefinitionBuilder
       .genericBeanDefinition(Config.class)
       .setScope(BeanDefinition.SCOPE_SINGLETON)
       .getBeanDefinition();
   beanFactory.registerBeanDefinition("config", configBeanDefinition);
   for (String name : beanFactory.getBeanDefinitionNames()) {
       // 不能识别注解注册的bean，打印结果没有 bean1
       System.out.println(name);
   }
   ```

3. 运行结果如下

   ![image-20220927001543342](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220927001543342.png)

4. 分析：打印结果没有bean1，可见beanFactory不能识别通过@Bean注册的bean

### BeanFactory后处理器

1. 添加一些BeanFactory后处理器

   ```java
   AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
   beanFactory
           // 获取bean工厂处理器
           .getBeansOfType(BeanFactoryPostProcessor.class)
           .values()
           // bean工厂处理器处理bean工厂
           .forEach(processor -> processor.postProcessBeanFactory(beanFactory));
   for (String name : beanFactory.getBeanDefinitionNames()) {
       System.out.println(name);
   }
   
   System.out.println(beanFactory.getBean(Bean1.class).getBean2());
   ```

2. 运行结果如下

   ![image-20220927002138652](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220927002138652.png)

3. 分析：可以打印出@Bean注册的bean1以及bean2了，但通过类Bean1调用getBean2方法并没有返回结果，所以自动注入的Bean还是不能获取到

### Bean后处理器

1. 添加Bean后处理器，针对bean的生命周期的各个阶段，如 @Autowired、@Resource

   ```java
   beanFactory
           .getBeansOfType(BeanPostProcessor.class)
           .values()
           .forEach(beanFactory::addBeanPostProcessor);
   System.out.println("--------------------");
   System.out.println(beanFactory.getBean(Bean1.class).getBean2());
   ```

2. 运行结果如下

   ![image-20220927002339191](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220927002339191.png)

3. 添加了Bean后处理器后，可以成功打印出bean2，注意到分隔符是在获取bean之前打印的，即BeanFactory并没有一开始就实例化所有的对象，而是按需加载

4. tips：如果没有注释前段的循环打印，那这一步添加的代码打印结果也为null，这是为什么？

### 初始实例化

1. 通过preInstantiateSingletons方法可以在加载BeanFactory时实例化所有的对象，修改代码为

   ```java
   beanFactory.preInstantiateSingletons();
   System.out.println("--------------------");
   System.out.println(beanFactory.getBean(Bean1.class).getBean2());
   ```

2. 运行结果如下：

   ![image-20220927002617161](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220927002617161.png)

3. 分析：可以看到在获取bean之前，BeanFactory就已经实例化所有的对象了

### 总结

1. 不会主动调用BeanFactory的后处理器
2. 不会主动添加Bean后处理器
3. 不会主动初始化单例
4. 不会解析beanFactory以及${}、#{}

### 拓展

#### Bean后处理器具有顺序

1. 新增接口Inter

   ```java
   interface Inter {
   }
   ```

2. 让Bean1和Bean2类都实现这个接口，并在Bean1类中注入一个Inter类对象

   ```java
   class Bean1 implements Inter {
       @Autowired
       @Resource(name = "bean1")
       Inter inter;
       public Inter getInter() {
           return bean1;
       }
   }
   ```

   ```java
   class Bean2 implements Inter {
   }
   ```

3. Config类中生命两个bean，一个为bean1，一个为bean2

   ```java
   @Configuration
   class Config {
       @Bean
       public Inter bean1() {
           System.out.println("开始构造bean1");
           return new Bean1();
       }
       @Bean
       public Inter bean2() {
           System.out.println("开始构造bean2");
           return new Bean2();
       }
   }
   ```

4. 问题：同时使用@Autowired和@Resource，输出是什么？

   ```java
   beanFactory
           .getBeansOfType(BeanPostProcessor.class)
           .values()
           .forEach(beanFactory::addBeanPostProcessor);
   System.out.println("--------------------");
   System.out.println(beanFactory.getBean(Bean1.class).getBean2());
   ```

5. 默认情况下是输出Bean2对象的，因为@Autowired具有更高的优先级

   ![image-20220927203719134](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220927203719134.png)

6. 通过beanFactory.getDependencyComparator()获取一个比较器，让@Resource具有更高的优先级

   ```java
   beanFactory
           .getBeansOfType(BeanPostProcessor.class)
           .values()
           .stream()
           .sorted(beanFactory.getDependencyComparator())
           .forEach(beanFactory::addBeanPostProcessor);
   System.out.println("--------------------");
   System.out.println(beanFactory.getBean(Bean1.class).getInter());
   ```

7. 此时输出为Bean2对象

   ![image-20220927203931065](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220927203931065.png)