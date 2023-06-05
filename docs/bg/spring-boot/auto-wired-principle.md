1. 从主程序类入手，查看@SpringBootApplication注解

   ![image-20220714001951068](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714001951068.png)

   该注解中一些元注解和3个核心注解

2. 查看@SpringBootConfiguration注解

   ![image-20220714002108598](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714002108598.png)

   该注解声明为配置类注解，也就标志着主程序类是一个配置类，回到1

3. 继续查看@EnableAutoConfiguration注解，

   ![image-20220714002254902](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714002254902.png)

   这个类有@AutoConfigurationPackage注解，并且导入一个类AutoConfigurationImportSelector.class

4. 查看@AutoConfigurationPackage

   ![image-20220714002434310](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714002434310.png)

   发现这个注解导入了类AutoConfigurationPackages.Registrar.class

5. 查看Registrar.class

   ![image-20220714002635829](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714002635829.png)

   通过断点，可以查看到此时(new PackageImports(metadata)).getPackageNames()的值为live.yangtao，即项目的默认包路径。

   ![image-20220713233442495](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220713233442495.png)

   而这个Registrar的功能就是给容器导入一系列组件，此时就是将默认包下的所有组件都导进来。

   ![image-20220714003043812](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714003043812.png)

6. 回到3，查看导入的类AutoConfigurationImportSelector.class，其中有一个方法为选择导入selectImports

   ![image-20220714003304785](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714003304785.png)

   获取自动导入选择器方法getAutoConfigurationEntry

   ![image-20220714003439760](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714003439760.png)

   获取候选配置方法getCandidateConfigurations

   ![image-20220714003702497](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714003702497.png)

   通过Spring工厂加载器的方法从指定路径加载指定类型工厂实现的全类名

   ![image-20220714003912017](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714003912017.png)

   具体的加载方法为

   ![image-20220714004104850](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714004104850.png)

   其中FACTORIES_RESOURCE_LOCATION的定义为

   ![image-20220714004149436](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714004149436.png)

   查看spring-boot-autoconfigure-2.5.0.jar\META-INF\spring.factories

   ![image-20220714004437470](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714004437470.png)

   总共有131个值，在获取候选配置中打断点，可看到SpringBoot启动时默认装配的组件就是上述文件中定义的自动装配组件

   ![image-20220714004951246](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714004951246.png)

7. 但是，这些组件并不是在项目启动时全部加载到容器中，而是根据条件装配注解，对组件按需加载，如AopAutoConfiguration是在存在Advice.class之后才会进行加载

   ![image-20220714013020859](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714013020859.png)

8. SpringBoot规范举例：在DispatcherServletAutoConfiguration类中，存在文件解析器MultipartResolver类，如下

   ![image-20220714230649702](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714230649702.png)

   ![image-20220714230818230](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714230818230.png)

   该类只有在引入MultipartResolver类后、且容器中不存在名称为multipartResolver的bean才会注册到容器中，然后在其参数中定义了一个MultipartResolver对象，并在方法最后将其返回，通过这种方式可以检查用户创建的文件解析器组件名称是否正确，如果不正确则返回一个解析器，且名称为multipartResolver（@Bean注解声明在方法上，则bean名称为方法名，即multipartResolver）。

9. SpringBoot与配置文件的绑定：以字符编码配置类HttpEncodingAutoConfiguration为例分析。

   HttpEncodingAutoConfiguration如下：

   ![image-20220714231720705](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714231720705.png)

   该类与ServerProperties类进行了配置绑定、要求原生Servlet模式、必须有CharacterEncodingFilter类（SpringWebMVC包中存在）、在配置文件中配置server.servlet.encoding后生效（无配置也匹配）。

   该类中还有另一个SSM项目中常用的组件CharacterEncodingFilter，@ConditionalOnMissingBean标志着如果用户没有自己注册这个组件，就会自动注册到容器中，这也标志着SpringBoot中以用户配置的组件优先。

   再看与HttpEncodingAutoConfiguration绑定的配置类ServerProperties

   ![image-20220714232542050](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714232542050.png)

   其中含有一个静态内部类Servlet

   ![image-20220714233001006](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714233001006.png)

   该静态内部类中含有属性encoding，类型为Encoding，Encoding类中定义了charset=UTF8

   ![image-20220714233202295](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714233202295.png)

   ![image-20220714233225885](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220714233225885.png)

10. SpringBoot配置定制化，以上可以看出，SpringBoot的配置类与配置文件互相绑定，因此在定制化这些配置的时候有两种选择：

   1. 在配置文件application.properties中修改自动配置类中导入的xxxProperties类所绑定的值，如

      ```properties
      server.servlet.encoding.charset=GBK
      ```

   2. 自定义一个bean，替换掉底层的组件，因为SpringBoot中用户自定义的组件优先级总在系统定义组件的优先级之上（@ConditionalOnMissBean），如：

      ```java
      @Configuration
      public class MyEncodingConfig {
          @Bean
          public CharacterEncodingFilter characterEncodingFilter() {
              CharacterEncodingFilter filter = new CharacterEncodingFilter();
              filter.setEncoding("GBK");
              // 拦截器设置的编码覆盖现有的编码（请求和响应）
              filter.setForceEncoding(true);
              return filter;
          }
      }
      ```

   ![image-20220715001304291](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220715001304291.png)
