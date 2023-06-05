## SpringBoot项目创建

1. 创建Maven项目，POM文件基本模板

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>live.yangtao</groupId>
       <artifactId>SpringBoot-01-HelloWorld</artifactId>
       <version>1.0-SNAPSHOT</version>
   
   </project>
   ```

2. 导入SpringBoot项目POM依赖，指定SpringBoot版本

   ```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.5.0</version>
   </parent>
   ```

3. Web开发场景依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   </dependencies>
   ```

4. SpringBoot入口类，编辑main方法

   ```java
   @SpringBootApplication
   public class SpringBoot01Application {
       public static void main(String[] args) {
           SpringApplication.run(SpringBoot01Application.class);
       }
   }
   ```

5. 在resource目录下创建项目的配置文件application.properties，指定服务的端口号

   ```properties
   server.port=8888
   ```

6. 编写一个controller

   ```java
   @RestController
   public class HelloController {
       @GetMapping("/hello")
       public String hello() {
           return "hello";
       }
   }
   ```

7. 运行入口类main方法

   ![image-20220711003508715](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711003508715.png)

8. 项目启动后，访问[localhost:8888/hello](http://localhost:8888/hello)，即可看到刚才编写的controller给我们返回的内容

   ![image-20220711003553731](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711003553731.png)

9. 通过jar包形式部署项目，需要在项目POM文件中指定打包方式以及maven插件

   ```xml
   ...
   <packaging>jar</packaging>
   
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ...
   ```

10. 通过Maven的package打包

    ![image-20220711004458899](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711004458899.png)

    ![](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711004543722.png)

11. 运行jar，即可启动项目

    ![image-20220711004803087](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220711004803087.png)

