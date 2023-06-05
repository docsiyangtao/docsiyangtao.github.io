## Jedis

### 使用步骤

1. 引入依赖

   ```xml
   <dependency>
       <groupId>redis.clients</groupId>
       <artifactId>jedis</artifactId>
       <version>3.7.0</version>
   </dependency>
   ```
2. 建立连接

   ```java
   public void init() {
       jedis = new Jedis("127.0.0.1", 6379);
       jedis.auth("");
       jedis.select(0);
   }
   ```
3. 操作

   ```java
   public void testString() {
       String res = jedis.set("name", "yangtao");
       System.out.println(res);
       String name = jedis.get("name");
       System.out.println(name);
   }
   ```
4. 释放连接

   ```java
   public void close() {
       if (jedis != null) {
           jedis.close();
       }
   }
   ```

## Jedis连接池

Jedis本身是线程不安全的，且频繁地创建和销毁连接会损耗性能，因此推荐使用Jedis连接池代替直连。

```java
public class JedisConnectionFactory {

    private static final JedisPool JEDIS_POOL;

    static {
        // 配置连接池
        JedisPoolConfig config = new JedisPoolConfig();
        // 最大连接数
        config.setMaxTotal(10);
        // 最大空闲连接数
        config.setMaxIdle(8);
        // 最小空闲连接数
        config.setMinIdle(0);
        // 无空闲连接时的等待时间
        config.setMaxWaitMillis(10000);
        // 创建连接池对象
        JEDIS_POOL = new JedisPool(config, "127.0.0.1", 6379, 1000);
    }

    public static Jedis getJedis() {
        return JEDIS_POOL.getResource();
    }

}
```

使用：

```java
// 获取连接
Jedis jedis = JedisConnectionFactory.getJedis();

// 释放连接
if (jedis != null) {
    jedis.close();
}
```



## SpringDataRedis

### 概述

SpringData对Redis的继承模块就叫做SpringDataRedis

* 提供了对不同Redis客户端的整合（Lettuce和Jedis）
* 提供了RedisTemplate统一操作Redis的API
* 支持Redis的发布订阅模式
* 支持Redis烧饼和Redis集群
* 支持基于Lettuce的响应式编程
* 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
* 支持基于Redis的JDKCollection实现

### API

| API                         | 返回类型        | 说明               |
| --------------------------- | --------------- | ------------------ |
| redisTemplate.opsForValue() | ValueOperations | 操作String类型数据 |
| redisTemplate.opsForHash()  | HashOperations  | 操作Hash类型数据   |
| redisTemplate.opsForList()  | ListOperations  | 操作List类型数据   |
| redisTemplate.opsForSet()   | SetOperations   | 操作Set类型数据    |
| redisTemplate.opsForZset()  | ZSetOperations  | 操作ZSet类型数据   |
| redisTemplate               |                 | 通用命令           |

### 使用步骤

1. 创建SpringBoot项目，引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-pool2</artifactId>
   </dependency>
   ```

2. 全局配置（序列化）

   默认的RedisTemplate使用了JDK序列化JdkSerializationRedisSerializer，会对字符进行转义，需手动设置key值使用String序列化器

   ```java
   @Configuration
   public class RedisConfig {
   
       @Bean
       public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
           // 创建RedisTemplate对象
           RedisTemplate<String, Object> template = new RedisTemplate<>();
           // 设置连接工厂
           template.setConnectionFactory(connectionFactory);
           // JSON序列化工具
           GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
           // 设置key的序列化
           template.setKeySerializer(RedisSerializer.string());
           template.setHashKeySerializer(RedisSerializer.string());
           // 设置value的序列化
           template.setValueSerializer(jsonRedisSerializer);
           template.setHashValueSerializer(jsonRedisSerializer);
           // 返回
           return template;
       }
   
   }
   ```

3. 通过RedisTemplate可以自动的进行序列化、反序列化

   ```java
   @Autowired
   private RedisTemplate<String, Object> redisTemplate;
   
   public void test1() {
       // 设置
       Map<String, Object> map = new HashMap<>();
       map.put("name", "yangtao");
       map.put("age", 22);
       map.put("address", "广州");
       redisTemplate.opsForValue().set("user", map);
       // 获取
       Object user = redisTemplate.opsForValue().get("user");
       System.out.println(user);
   }
   ```

4. 另一种方法是直接注入StringRedisTemplate，键值都是用String序列化器，手动对值进行序列化、反序列化

   ```java
   @Autowired
   private StringRedisTemplate stringRedisTemplate;
   
   public void test2() {
       Map<String, Object> map = new HashMap<>();
       map.put("name", "yangtao2");
       map.put("age", 23);
       map.put("address", "广州天河");
       stringRedisTemplate.opsForValue().set("user", JSON.toJSONString(map));
       String user = stringRedisTemplate.opsForValue().get("user");
       System.out.println(JSON.parseObject(user, Map.class));
   }
   ```

