## 系统级

```bash
# 启动
redis-server xxx.conf

# 进入redis自带客户端
redis-cli -h 127.0.0.1 -p 6379 -a pwd

# 关闭连接
shutdown

# 测试连通性
ping

# 获取帮助命令
help @generic
```



## 压力测试

```bash
redis-benchmark [options] [value]
	-h：指定服务器主机名
	-p：指定服务器端口
	-s：指定服务器socket
	-c：指定并发连接数
	-n：指定请求数
	-d：以字节的形式指定set/get值的数据大小
	-r：set/get/incr使用随机key，sadd使用随机值
	-P：通过管道传输请求
	-q：强制退出redis，仅显示query/sec值
	--csv：以CSV格式输出
	-l：生成循环，永久执行测试
	-t：仅运行以逗号分隔的测试命令列表
	-I：Idle模式，仅打开N个idle连接并等待
	
# 100个并发连接，100000个请求，检测host为localhost，端口为6379的redis服务器
redis-benchmark -h localhost -c 100 -n 100000 -p 6379
```



## 数据库

```bash
# 选择数据库
select <num>

# 查看当前数据库的key数量
dbsize

# 清空当前数据库
flushdb

# 清空所有数据库
flushall
```



## 键key

```bash
# 设置键值对
set key value

# 获取key对应的值
get key

# 删除key
del key

# 查看所有key
keys *

# 查看是否存在key
exists key

# 将key转移到指定的数据库
move key index

# 设置过期时间
expire key second

# 查看key的存活时间
ttl key

# 查看key的类型
type key
```



## 字符串String

```bash
# 设置键值对
set key value

# 获取key对应的值
get key

# 设置多个键值对
mset key value [key value...]

# 获取多个值
mget key [key]

# 删除key
del key

# key是否存在
exists key

# 若key不存在，则相当于set；若key存在，则相当于append
append key value

# 获取key的字符串长度
strlen key

# 将key对应的值自增（Integer）
incr key

# 将key对应的值自减（Integer）
decr key

# 将key对应的值增加value（Integer）
incrby key incrment

# 将key对应的值减少value（Integer）
decrby key decrment

# 浮点数增长（需指定增长数）
incrbyfloat key incrment

# 获取key下标[start,end]对应的内容
getrange key start end

# 获取全部的值
getrange key 0 -1

# 设置key指定下标offset的内容为value
setrange key offset value

# 设置key的过期时间
setex key seconds value

# 如果key不存在则设置键值对
setnx key value

# 设置多个值，原子性操作
msetnx key value [key value...]

# 设置key，返回上一次设置的value
getset key value
```



## 列表List

```bash
# 往列表头部添加一个或多个值
lpush key value [value...]

# 往列表尾部添加一个或多个值
rpush key value [value...]

# 获取列表范围内的值[start,stop]
lrange key start stop

# 获取列表的所有值
lrange key 0 -1

# 弹出列表头
lpop key

# 弹出列表尾
rpop key

# 获取列表指定下标的值
lindex key index

# 获取列表的长度
llen key

# 移除count个与value相等的值
lrem key count value

# 修剪，保留[start,stop]的值
ltrim key start stop

# 将sourc的最后一个值弹出添加到destination
rpoplpush source destination

# 将列表下标为index设置为value
lset key index value

# 在key前面|后面插入value
linsert key before|after pivot value

# 阻塞获取元素（未获取到等待一段时间）
blpop / brpop key [key ...] timeout
```



## 集合Set

```bash
# 将一个或多个元素添加到集合中
sadd key member [member...]

# 返回集合中的所有成员
smenbers key

# member是否是key的成员之一
sismember key member

# 获取集合里面的元素个数
scard key

# 移除集合中的一个或多个元素
srem key member [member...]

# 随机返回集合中的n个元素
srandmember key [count]

# 随机弹出集合中的n个元素
spop key [count]

# 将成员从一个集合移到另一个集合
smove source destination member

# 返回集合2中没有的集合1元素
sdiff key1 [key2...]

# 返回两个集合的交集
sinter key key

# 返回两个集合的并集
sunion key key
```



## 哈希Hash

```bash
# 设置键值对 key - <field,value>
hset key field value

# 获取哈希表中的的field字段对应的值
hget key field

# 存入多个
hmset key field value

# 获取多个
hmget key field

# 获取哈希表中所有的键值对
hgetall	key

# 删除哈希表中一个或多个字段
hdel key field [field]

# 获取哈希表的键值对的数量
hlen key

# 哈希表中是否存在字段
hexists key field

# 哈希表中所有的字段
hkeys key

# 哈希表中所有的值
hvals key

# 为哈希表中的字段增加指定值
hincrby key field incrment

# 为哈希表中不存在的字段赋值
hsetnx key field value
```



## 有序集合Zset

```bash
# 将一个或多个值及其分值添加到须有序集合中
zadd key score member

# 获取指定元素的score值
zscore key member

# 返回指定区间内的成员（显示分值并递增）
zrange key start stop [withscores]

# 显示指定区间的集合
zrangebyscore key min max [withscores] [limit]

# 显示正无穷到负无穷的成员
zrangebyscore key -inf +inf

# 移除集合中的一个或多个成员
zrem key member [member...]

# 返回集合的成员数量
zcard key

# 统计score值在给定范围内的所有元素的个数
scount key min max

# 返回集合中成员从小到大的排位
zrank key member

# 返回集合中成员从大到小的排位
zrevrank key member

# 返回集合2中没有的集合1元素
zdiff key1 [key2...]

# 返回两个集合的交集
zinter key key

# 返回两个集合的并集
zunion key key
```



## 地理位置GEO

```bash
# 添加地理位置，值为经纬度
geoadd key longitudu latitude member

# 返回key里指定位置的经纬度
geopos key member [member...]

# 返回两地的距离
geodist key member1 member2 [unit]

# 给定经纬度为中心，找出某一半径内的元素
georadius key longitude latitude radius m|km|

# 返回位置名称和中心距离
georadius key longitude latitude radius m|km| withdist

# 返回位置名称和经纬度
georadius key longitude latitude radius m|km| withcoord

# 返回位置名称距离和经纬度，限定个数为n
georadius key longitude latitude radius m|km withdist withcoord count n

# 找出指定范围内的元素，中心点由给定的位置元素决定
georadiusbymember key member radius m|km|ft|m [withcoord] [withdist] [withhash] [asc|desc] [count count]

# 经纬度转化为字符串，字符串越长表示位置更精确，两个字符串越相似距离越近
geohash key member [member...]

# geo并没有删除，底层通过zset实现，因此用zrem
zrem key member [member...]
```



## HyperLogLog

```bash
# 添加元素到HyperLogLog中
pfadd key element [element...]

# 返回技术估算值
pfcount key [key...]

# 将多个HyperLogLog合并为一个
pfmerge destkey sourcekey [sourcekey...]
```



## BitMap

```bash
# 设置key的第offset位位1或0
setbit key offset value

# 获取offset位的值
getbit key offset

# 查看key上位为1的个数
bitcount key [start end]
```

