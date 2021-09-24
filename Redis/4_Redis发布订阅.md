## Redis发布订阅

Redis 发布订阅（pub/sub）是一种消息通信模式，发布者(pub)发送消息，订阅者(sub)接受消息

Redis客户端可以订阅任意数量的频道

#### 发布订阅关系

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

<img src="D:%5CTypora%5Cworkspace%5Cimg%5Cpubsub1.png" alt="img"  align='left'/>

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

<img src="D:%5CTypora%5Cworkspace%5Cimg%5Cpubsub2.png" alt="img"  align='left'/>

#### 示例

cli-1

```bash
# SUBSCRIBE channel [channel ...]
# Listen for messages published to the given channels
127.0.0.1:6379> subscribe LLL		#订阅频道 LLL
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "LLL"
3) (integer) 1
#等待消息。。。。
#频道中有消息发布，就会收到消息
1) "message"	#收到message
2) "LLL"		#频道名称
3) "hello"		#内容
1) "message"	
2) "LLL"
3) "123"
```

cli-2

```bash
# PUBLISH channel message
# Post a message to a channel
127.0.0.1:6379> publish LLL hello	#向频道 LLL 发送消息
(integer) 1
127.0.0.1:6379> publish LLL 123
(integer) 1
```

发布订阅相关命令

| 命令                                                         | 描述                               |
| ------------------------------------------------------------ | ---------------------------------- |
| [PSUBSCRIBE](https://www.redis.com.cn/commands/psubscribe.html) | 订阅一个或多个符合给定模式的频道。 |
| [PUBSUB](https://www.redis.com.cn/commands/pubsub.html)      | 查看订阅与发布系统状态。           |
| [PUBLISH](https://www.redis.com.cn/commands/publish.html)    | 将信息发送到指定的频道。           |
| [PUNSUBSCRIBE](https://www.redis.com.cn/commands/punsubscribe.html) | 退订所有给定模式的频道。           |
| [SUBSCRIBE](https://www.redis.com.cn/commands/subscribe.html) | 订阅给定的一个或多个频道的信息。   |
| [UNSUBSCRIBE](https://www.redis.com.cn/commands/unsubscribe.html) | 指退订给定的频道。                 |

#### 底层原理

redis-server 里维护了一个字典，字典的 key 就是 频道，字典的值是一个链表，链表中保存了所有订阅这个频道的 客户端

SUBSCRIBE 命令就是将客户端加入到这个 channel 的订阅链表中

PUBLISH 命令会找到 channel 对应的key，遍历链表，将 message 发送给所有的订阅者