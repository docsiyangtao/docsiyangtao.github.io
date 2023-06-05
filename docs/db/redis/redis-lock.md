JVM外部，多JVM共享同一个锁监视器，这样，无论是JVM内部的线程，还是多个JVM下不同的线程，都可以实现互斥的操作同一资源

分布式锁：满足分布式系统或集群模式下多进程可见并且互斥的锁。

实现方案：

1. redis
2. MySQL

难点：

1. 互斥
2. 高可用
3. 高性能
4. 安全性
5. 功能性



分布式锁比较

|        | MySQL                            | Redis                       | Zookeeper                          |
| ------ | -------------------------------- | --------------------------- | ---------------------------------- |
| 互斥   | 利用MySQL本身的互斥锁机制        | 利用setnx这样的互斥命令     | 利用节点的唯一性和有序性实现互斥   |
| 高可用 | 较好（主从）                     | 好（主从、集群）            | 好（集群）                         |
| 高性能 | 一般（基于磁盘IO）               | 好（基于内存）              | 一般（强一致性，需牺牲一定的性能） |
| 安全性 | 事务机制（自动释放锁、数据回滚） | 一般，只能利用key的过期机制 | 临时节点，断开连接自动释放         |



基于Redis的分布式锁

<img src="https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20221129003818776.png" alt="image-20221129003818776" style="zoom: 67%;" />

两个基本方法：

1. 获取锁，需要保证互斥性，确保只有一个线程获取锁，过期时间不能太短，需要保证设置锁和设置超时时间的操作是原子操作

   ```bash
   setnx lock thread1 nx ex 10
   ```

2. 释放锁，正常情况下手动释放锁，需考虑到服务宕机的情况，使用超时释放保证安全

   ```bash
   del lock
   ```

实现

```java
@Component("SimpleRedisLockUtil")
public class SimpleRedisLockUtil implements Lock {

    private static final String SIMPLE_REDIS_LOCK_PREFIX = "SIMPLE_REDIS_LOCK:";

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public boolean tryLock(String key, long timeout) {
        long threadId = Thread.currentThread().getId();
        return Boolean.TRUE.equals(stringRedisTemplate
                .opsForValue()
                .setIfAbsent(SIMPLE_REDIS_LOCK_PREFIX + key, 
                             threadId + "", 
                             timeout, 
                             TimeUnit.SECONDS));
    }

    @Override
    public void unlock(String key) {
        stringRedisTemplate.delete(SIMPLE_REDIS_LOCK_PREFIX + key);
    }
}
```

业务调用

```java
@GetMapping("/test1")
public void test1(@RequestParam("id") String id) {
    String lockKey = "test1_" + id;
    boolean lock = redisLockUtil.tryLock(lockKey, 100);
    if (lock) {
        try {
            // 线程休眠5S模拟业务处理
            Thread.sleep(5000);
            logger.info("id {} 业务执行成功", id);
        } catch (Exception e) {
            logger.error("id {} 业务执行出错：", id, e);
        } finally {
            redisLockUtil.unlock(lockKey);
        }
    } else {
        throw new RuntimeException("id " + id + " 有相同业务正在执行");
    }
}
```



获取

1. 阻塞式：获取不到时对CPU压力比较大，实现麻烦
2. 非阻塞式：只尝试一次，失败就立即返回不等待



可能遇到的问题：

1. 线程1遇到阻塞，导致锁超时释放了，但此时线程2已经进来了，并且获取到了锁，开始执行业务，此时线程1执行完成了，开始释放锁，但因为两个线程获取锁的标识相同，所以线程1把线程2的锁给释放掉了，当线程2执行完成的时候释放锁就遇到了无锁可以释放的问题，此时就出现问题了

   ![image-20221129004023109](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20221129004023109.png)

   解决：释放锁的时候先去判断值value，是否为当前线程加的锁，是当前线程的锁才去释放锁

   ![image-20221129004329174](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20221129004329174.png)

   实现

   <img src="https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20221129004437028.png" alt="image-20221129004437028" style="zoom:67%;" />

   ```java
   @Component("SimpleRedisLockUtil")
   public class SimpleRedisLockUtil implements Lock {
   
       private static final String SIMPLE_REDIS_LOCK_PREFIX = "SIMPLE_REDIS_LOCK:";
   
       // 作为线程锁的值
       private static final String UID = UUID.randomUUID().toString().replace("-", "");
   
       @Resource
       private StringRedisTemplate stringRedisTemplate;
   
       @Override
       public boolean tryLock(String key, long timeout) {
           // long threadId = Thread.currentThread().getId();
           return Boolean.TRUE.equals(stringRedisTemplate
                   .opsForValue()
                   .setIfAbsent(SIMPLE_REDIS_LOCK_PREFIX + key, 
                                UID, 
                                timeout, 
                                TimeUnit.SECONDS));
       }
   
       @Override
       public void unlock(String key) {
           // redis中的锁值
           String redisValue = stringRedisTemplate
               .opsForValue()
               .get(SIMPLE_REDIS_LOCK_PREFIX + key);
           // 当前线程的锁值
           String currentThreadValue = UID;
           // 相同才进行删除
           if (Objects.equals(redisValue, currentThreadValue)) {
               stringRedisTemplate.delete(SIMPLE_REDIS_LOCK_PREFIX + key);
           } else {
               throw new RuntimeException("超时");
           }
       }
   }
   ```

2. 上述1中判断锁和释放锁是两个操作，并不是原子操作，如果在两个操作的中间，产生了阻塞（如垃圾回收），此时另一线程拿到了锁，执行完业务并释放了锁，之后线程1阻塞结束，因为经过了判断不会再判断，直接释放了锁，出现问题

   ![image-20221129011315859](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20221129011315859.png)

   解决：保证判断锁和释放锁是一个原子操作

   Lua脚本

   Redis提供了Lua脚本功能，在一个脚本中可以编写多条Redis命令，保证多条命令执行时的原子性。

   流程

   1. 获取锁中的线程标识
   2. 判断是否与指定的标识一致
   3. 如果一致则释放锁（删除KEY，返回1）
   4. 如果不一致则不做操作（返回0）

   ```lua
   -- 两个参数：KEYS[1]、ARGV[1]
   if (redis.call('GET', KEYS[1]) == ARGV[1])) then
       return redis.call('DEL', KEYS[1])
   end
   return 0
   ```

   \* redis中执行lua脚本

   ```lua
   EVAL script numkeys key [key ...] arg [arg ...]
   ```

   在微服务中执行Lua脚本

   ```java
   // 加载类时就读取Lua脚本，提高效率
   private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;
   
   static {
       UNLOCK_SCRIPT = new DefaultRedisScript<>();
       UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));
       UNLOCK_SCRIPT.setResultType(Long.class);
   }
   
   @Override
   public void unlock(String key) {
       // 将判断以及删除交给redis执行，微服务只获取结果，从这个层面上看，这是一个原子操作
       Long res = stringRedisTemplate.execute(
           UNLOCK_SCRIPT,
           Collections.singletonList(SIMPLE_REDIS_LOCK_PREFIX + key),
           UID);
       if (Objects.equals(res, 0L)) {
           throw new RuntimeException("超时");
       }
   }
   ```

