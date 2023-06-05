1. 查看SpringBoot父项目依赖spring-boot-starter-parent-2.5.0.pom，再点进spring-boot-dependencies，里面规定了各种依赖的版本号，引入了各类依赖

   ![image-20220711005648412](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711005648412.png)

   ![image-20220711005830805](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711005830805.png)

2. 场景启动器

   如spring-boot-starter-web，就是web开发的场景依赖包含SpringMVC，还有其他的如spring-boot-starter-data-jdbc、spring-boot-starter-data-redis等等，所有的官方依赖可见https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters

3. 版本依赖树

   ![image-20220711010449710](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711010449710.png)