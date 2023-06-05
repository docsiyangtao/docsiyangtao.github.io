## 传统配置绑定

传统方式都是通过java的IO流来获取文件（如properties）中的内容，及其麻烦，如下：

```properties
server.port=8888
```

```java
public class MyProperties {

    private static String serverPort;

    static {
        Properties properties = new Properties();
        Class<MyProperties> myPropertiesClass = MyProperties.class;
        ClassLoader classLoader = myPropertiesClass.getClassLoader();
        InputStream resourceAsStream = classLoader.getResourceAsStream("application.properties");
        try {
            properties.load(resourceAsStream);
        } catch (IOException e) {
            log.error("加载配置文件出错");
        }
        Field[] fields = myPropertiesClass.getDeclaredFields();
        Arrays.stream(fields).forEach(field -> {
            Object value = properties.getOrDefault(humpNameHandle(field.getName()), null);
            field.setAccessible(true);
            try {
                field.set(myPropertiesClass, value);
            } catch (IllegalAccessException e) {
                log.error("属性 {} 赋值出错", field.getName(), e);
            }
        });
    }

    private static String humpNameHandle(String humpName) {
        if (humpName == null) {
            return null;
        }
        StringBuilder res = new StringBuilder();
        byte[] bytes = humpName.getBytes();
        for (byte aByte : bytes) {
            res.append((aByte >= 65 && aByte <= 90) ? "." + (char) aByte : (char) aByte);
        }
        return res.toString().toLowerCase();
    }

    public static void main(String[] args) {
        System.out.println(MyProperties.serverPort);	// 8888
    }

}
```

这种方式过于繁琐，且需要手动将配置文件的内容与类中的属性进行绑定。



## SpringBoot配置绑定

在配置文件中声明如下配置

```properties
car.benz.level=S
car.benz.price=1880000
```

1. 声明组件 + 配置前缀

   将当前Properties类声明为一个组件，并通过@ConfigurationProperties注解指定配置的前缀名称

   ```java
   @Component("benzProperties")
   @ConfigurationProperties(prefix = "car.benz")
   @Data
   public class BenzProperties {
       private String level;
       private double price;
   }
   ```

2. 配置前缀 + 绑定配置

   当前Properties类指定配置的前缀名称，其他配置类通过@EnableConfigurationProperties，将Properties类与配置文件中的配置进行绑定，将Properties类注册到容器中。对比方式一，此方式可以通过配置类将Properties注册到容器中

   ```java
   @ConfigurationProperties(prefix = "car.benz")
   @Data
   public class BenzProperties {
   
       private String level;
       private double price;
   
   }
   ```

   ```java
   @Configuration
   @EnableConfigurationProperties(BenzProperties.class)
   public class BenzConfig {
   }
   ```

