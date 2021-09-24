## Redis

NoSQL分类：

- KV键值对：Redis
- 文档型：MongoDB

Redis 是一种基于**内存**的NoSQL数据库，支持**集群、分布式、主从同步**

### 1. 安装

windows下载地址：[Releases · microsoftarchive/redis (github.com)](https://github.com/MicrosoftArchive/redis/releases)

linux 官网有

下载后直接解压，可以找到 **redis-server.exe** 和 **redis-cli.exe** 对应服务器端和客户端

启动服务器端，再启动客户端，默认端口号为 6379

**redis-benchmark.exe** 为压力测试

### 2. 基础数据类型

redis.conf 中可以看到 databases 16	默认16个数据库，默认使用的是第0个

#### 2.1基本命令

```bash
127.0.0.1:6379> select 1	#选择1号数据库
OK
127.0.0.1:6379[1]> dbsize	#查看数据库大小
(integer) 0
127.0.0.1:6379[1]> set name LLL	#set key value
OK
127.0.0.1:6379[1]> get name	#get key
"LLL"
127.0.0.1:6379[1]> keys *	#keys pattern *表示任意, ?表示一个占位
1) "name"
127.0.0.1:6379> exists name	#查看是否存在key
(integer) 1
127.0.0.1:6379[1]> flushdb	#清空当前数据库数据
OK
127.0.0.1:6379[1]> flushall	#清空所有数据库数据
OK
127.0.0.1:6379> move name 1	#move key to db 把name这个key移到数据库1
OK
127.0.0.1:6379> expire age 3	#设置当前key的过期时间
(integer) 1
127.0.0.1:6379> ttl age		#查看当前key的剩余时间
(integer) -2
```

Redis 是单线程的，所有数据都存放在内存中，所以用单线程操作效率最高，多线程存在上下文切换耗时

#### 2.2 String (K-V)

setnx在分布式锁中常用，一次设置多个键值对时，如果有一个已经存在就所有的键值对都设置失败

```bash
127.0.0.1:6379> append name 123	#向当前key的value后拼接字符串
(integer) 6
127.0.0.1:6379> strlen name		#查看当前key的长度
(integer) 6
127.0.0.1:6379> set age 1		#age存储的是integer类型
OK
127.0.0.1:6379> incr age		# +1
(integer) 2
127.0.0.1:6379> decr age		# -1
(integer) 1
127.0.0.1:6379> incrby age 10	# incrby key 增量
(integer) 11
127.0.0.1:6379> getrange name 0 1	#截取字符串的[0,1]
"Ll"
127.0.0.1:6379> getrange name 0 -1	#截取字符串的所有
"LlL123"
127.0.0.1:6379> setex sex 10 1	# SETEX key seconds value 设置sex的value 10秒过期
OK								# Set the value and expiration of a key
127.0.0.1:6379> setnx name WWW	# Set the value of a key, only if the key does not exist
(integer) 0						# 在分布式锁中常用

127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3	# 一次设置多个值
OK
127.0.0.1:6379> mget k1 k2 k3	# 一次获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k5 v5	# k1 v1已经存在，所以整个失败，原子操作
(integer) 0
127.0.0.1:6379> getset name WWW		# 先get 再set
"LlL123"
127.0.0.1:6379> get name
"WWW"
```

#### 2.3 List（K-list）

本身是一个链表，可以用来做 队列 栈

```bash
127.0.0.1:6379> lpush L1 1 2 3	#从左边向list中赋值==> 3 2 1
(integer) 3
127.0.0.1:6379> lpop L1		#从左边弹出值==> 3 - 2 1
"3"
127.0.0.1:6379> lpop L1		#从左边弹出值==> 2 - 1
"2"
127.0.0.1:6379> lpop L1		#从左边弹出值==> 1
"1"
127.0.0.1:6379> lrange L1 0 -1	#lrange读取的也是从左边开始数
1) "3"
2) "2"
3) "1"
127.0.0.1:6379> rpush L1 1 2 3 4	#从右边赋值
(integer) 4
127.0.0.1:6379> rpop L1				#从右边弹出值
"4"

127.0.0.1:6379> lrange L1 0 -1		#L1目前有的值
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> lindex L1 0			#获取L1从左数下标为index的值
"1"
127.0.0.1:6379> llen L1				#返回列表长度
(integer) 3
127.0.0.1:6379> lrem L1 2 1			# LREM key count value 删除列表中指定个数的value
(integer) 2
127.0.0.1:6379> ltrim L1 1 3		#截取L1中下标在[1,3]中的值，会改变原list
OK
127.0.0.1:6379> lset L1 0 x			# LSET key index value 向指定下标设置值，像数组(列表必须存在)
OK

127.0.0.1:6379> lpush L1  1 2 3 4		# L1 : 4 3 2 1
(integer) 4
127.0.0.1:6379> linsert L1 before 1 1.1	# LINSERT key BEFORE|AFTER pivot value
(integer) 5
127.0.0.1:6379> lrange L1 0 -1			# 可以看到1之后插入了1.1
1) "4"
2) "3"
3) "2"
4) "1.1"
5) "1"
127.0.0.1:6379> linsert L1 after 1 0.9	# 1之前插入了0.9
(integer) 6
127.0.0.1:6379> lrange L1 0 -1
1) "4"
2) "3"
3) "2"
4) "1.1"
5) "1"
6) "0.9"
```

#### 2.4 Set（不重复的K-V）

集合，无重复元素

```bash
127.0.0.1:6379> sadd set1 v1 v2	# 向集合中添加值
(integer) 2
127.0.0.1:6379> sadd set1 v1 v3	# 添加一个重复值和不重复值，只能成功不重复的
(integer) 1
127.0.0.1:6379> scard set1		# 查看集合中元素个数
(integer) 3
127.0.0.1:6379> smembers set1	# 查看集合中所有的元素
1) "v3"
2) "v2"
3) "v1"
127.0.0.1:6379> srem set1 v2	# 删除指定set中的指定元素
(integer) 1
127.0.0.1:6379> spop set1		# 随机移除集合中的元素
"v3"
127.0.0.1:6379> srandmember set1	# 返回set中的一个随即元素
"v3"

127.0.0.1:6379> sadd set1 v1 v2 v3	# 集合1
(integer) 3
127.0.0.1:6379> sadd set2 v3 v4 v5	# 集合2
(integer) 3
127.0.0.1:6379> sdiff set1 set2		# 求集合1、2的差集(集合1中有,集合2中没有的)
1) "v1"
2) "v2"
127.0.0.1:6379> sinter set1 set2	# 求集合1、2的交集
1) "v3"
127.0.0.1:6379> sunion set1 set2	# 求集合1、2的并集
1) "v1"
2) "v2"
3) "v3"
4) "v5"
5) "v4"
```

#### 2.5 Hash（K-Map）

适合对象的存储，如 user  name  Lll  age  18

```bash
127.0.0.1:6379> hset map1 feild1 value1					#向map中设置一个键值对
(integer) 1
127.0.0.1:6379> hget map1 feild1						#从map中获取一个键的值
"value1"
127.0.0.1:6379> hmset map1 feild1 value1 feild2 value2	#向map中一次设置多个键值对
OK
127.0.0.1:6379> hmget map1 feild1 feild2				#从map中一次获取多个键的值
1) "value1"
2) "value2"

127.0.0.1:6379> hgetall map1		#获取map中所有的键值对
1) "feild1"
2) "value1"
3) "feild2"
4) "value2"
127.0.0.1:6379> hdel map1 feild1	#删除map中指定k-v对
(integer) 1
127.0.0.1:6379> hgetall map1
1) "feild2"
2) "value2"
127.0.0.1:6379> hlen map1			#查看当前map中有几个键值对
(integer) 1

127.0.0.1:6379> hkeys map1	# 获取map中的所有key
1) "k1"
2) "k2"
3) "k3"
127.0.0.1:6379> hvals map1	# 获取map中的所有value
1) "v1"
2) "v2"
3) "v3"

127.0.0.1:6379> hexists map1 k1	#判断当前map中是否存在指定key
(integer) 1
```

#### 2.6 Zset（有序集合）

适用于

- 存储需要排序的数据
- 设置权限级别，比如 1为最高权限，2。。。，可以带权重进行判断

```bash
127.0.0.1:6379> zadd set1 1 v1 2 v2 3 v3	# ZADD key score member 其中score相当于权重
(integer) 3
127.0.0.1:6379> zrangebyscore set1 -inf inf # 根据权值排序，-inf和inf是最小值到最大值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> zrevrangebyscore set1 inf -inf	#根据权值倒序排序
1) "v3"
2) "v2"
3) "v1"
127.0.0.1:6379> zcount set1 1 2	#计算区间score区间内的元素个数
(integer) 2

127.0.0.1:6379> zrem set1 v1	#移除元素
(integer) 1
```

### 3. 特殊数据类型

#### 3.1 Geospatial地理位置

底层实现为Zset，所以zset的命令也可以操作

```bash
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing	
#GEOADD key longitude latitude member [longitude latitude member ...]
127.0.0.1:6379> geopos china:city beijing
#从key里返回所有给定位置元素的位置（经度和纬度）
127.0.0.1:6379> geodist china:city beijing shanghai m
#GEODIST key member1 member2 [unit]  返回两个地理位置之间的距离，unit为m、km、mi(英里)、ft(英尺)
127.0.0.1:6379> georadius 110 30 1000 km
#GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
#以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
```

#### 3.2 HyperLogLog

> 基数

A{1,2,3,4,5}

B{1,3,5,7,9}

基数（不重复元素个数）=4

Redis HyperLogLog就是做基数统计的数据结构

优点：内存固定，占内存小

> 应用
>
> 网页的访问人数，一个用户访问多次，也算做一个人
>
> - 传统做法：用set集合保存用户id，统计set中元素数量即可	缺点：目的是为了求数量，却保存了大量用户id，浪费空间
> - 使用HyperLogLog统计，只计算数量

```bash
127.0.0.1:6379> pfadd key1 a b c d	# 向key中添加不重复元素
(integer) 1
127.0.0.1:6379> pfcount key1		# 统计key中的元素数量
(integer) 4
127.0.0.1:6379> pfadd key2 b e f	
(integer) 1
127.0.0.1:6379> pfcount key2
(integer) 3
127.0.0.1:6379> pfmerge key3 key1 key2	#合并两个集合，只保留不重复的项
OK
127.0.0.1:6379> pfcount key3
(integer) 6
```

####  3.3 Bitmap

位存储，适用于只有两个状态的存储

统计用户信息：活跃/不活跃、登录/未登录

```bash
127.0.0.1:6379> setbit k1 0 1	#设置第0位为1
(integer) 0
127.0.0.1:6379> setbit k1 1 0	#设置第1位为0
(integer) 0
127.0.0.1:6379> setbit k1 2 1	#设置第2位为1
(integer) 0
127.0.0.1:6379> bitcount k1		#统计k1中为1的位数
(integer) 2
```

### 4. Redis事务

事务就是一次性执行多个命令，Redis事务保证：

- 所有命令都会被序列化，按照顺序执行，执行过程中不会被打断
- 是一个原子操作，一个事务中命令要么都被执行，要么都不执行

Redis事务没有隔离级别的概念

所有命令在事务中没有直接被执行，只有发起EXEC命令时才被执行

- 开启事务（multi）
- 命令入队
- 执行事务（exec）

Redis单条命令保证原子性，事务不保证原子性！（因为不会回滚）

#### 4.1 正常执行事务

```bash
127.0.0.1:6379> multi	#开启事务
OK
#命令入队
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> exec	#执行事务
1) OK
2) OK
```

#### 4.2 放弃事务

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> discard		#放弃事务
OK
127.0.0.1:6379> get k3		#当前事务队列中的命令都不会被执行
(nil)
```

#### 4.3 异常事务

> 编译时错误，排队命令发生错误，放弃事务

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a 1
QUEUED
127.0.0.1:6379> lget a		#错误语句
(error) ERR unknown command 'lget'
127.0.0.1:6379> exec		#事务已经被放弃
(error) EXECABORT Transaction discarded because of previous errors.
```

> 运行时错误，EXEC之后发生的错误，即使有些命令发生错误，其他命令也会执行

```bash
127.0.0.1:6379> multi		#开启事务
OK
127.0.0.1:6379> set k1 v5	#写入String类型
QUEUED
127.0.0.1:6379> lpop k1		#对String类型的数据进行list操作
QUEUED
127.0.0.1:6379> exec		#队列中第二条语句执行报错
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1		#同一个事务中的其他命令仍然被执行了！
"v5"
```

### 5. Redis实现乐观锁

> 乐观锁：认为不会出错，所以不上锁，更新时先去判断一下是否有人修改了数据，例如mysql的MVCC
>
> - 获取version
> - 更新时比较version

**WATCH**命令

可以当作Redis的乐观锁操作

- 执行exec时会先去比对watch命令监控的键值对
- 如果没有被改变，就执行命令
- 如果发生改变，就回滚事务
- 取消事务前的watch命令

客户端1，监视了a的值，然后入队修改a的命令，还未执行EXEC时客户端2修改了a的值，则EXEC失败

```bash
127.0.0.1:6379> set a 100
OK
127.0.0.1:6379> set b 0
OK
127.0.0.1:6379> watch a		#监视a这个字段
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a 90
QUEUED
127.0.0.1:6379> set b 10	
QUEUED
127.0.0.1:6379> exec		#执行exec前，其他客户端修改了a的数据，则执行失败！
(nil)
```

客户端2，修改a的值

```bash
127.0.0.1:6379> get a
"100"
127.0.0.1:6379> set a 1000
OK
```

