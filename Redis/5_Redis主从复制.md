## Redis主从复制

主从复制的作用：

- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
- 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速故障恢复，实际上是一种服务冗余
- 负载均衡：主从复制 + 读写分离
  - 主机处理写请求，从机处理读请求，可以减少服务器的压力
  - 尤其是在写少读多的情况下，通过多个从节点负担读负载，可以大大提高 Redis 服务器的并发量
- 高可用：哨兵和集群的基础

### 配置主从机

最少为 一主二从

1. 复制三份 redis.conf 文件
2. 修改每份配置文件
   - 修改端口号
   - 修改 pidfile 名字
   - 修改 logfile 名字
   - 修改dump.rdb 名字

默认情况下，每台 Redis 服务器都是主节点，所以只需要配置从机就行了

```bash
127.0.0.1:6379> info replication	#查看节点主从复制信息
# Replication
role:master			#角色
connected_slaves:0	#从机数
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

在从机中进行配置

- 命令的方式，暂时的

```bash
127.0.0.1:6380> slaveof 127.0.0.1 6379	#将这个节点设置为另一个节点的从机
# SLAVEOF host port
# summary: Make the server a slave of another instance, or promote it as master
```

- 配置文件中修改，永久的

```bash
slaveof <masterip> <masterport>
#<masterip>   主机ip
#<masterport> 主机端口号
```

 没有哨兵模式时

- 主机宕机，从机中还有数据，从机如果想要成为主机，可以使用 SLAVEOF no one 变成 master，其他从机就连接在它之下
- 从机宕机，再重新启动，会立刻从主机中复制数据（全量复制）

### 复制原理

#### 全量同步

Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份。具体步骤如下： 

- 从服务器连接主服务器，发送SYNC命令； 
- 主服务器接收到SYNC命名后，开始执行BGSAVE命令生成RDB文件并使用缓冲区记录此后执行的所有写命令； 
- 主服务器BGSAVE执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令； 
- 从服务器收到快照文件后丢弃所有旧数据，载入收到的快照； 
- 主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令； 
- 从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

<img src="D:%5CTypora%5Cworkspace%5Cimg%5Cimage-20210923201351417.png" alt="image-20210923201351417" style="zoom:75%;" align='left'/>

#### 增量同步

Slave 正常工作时 主服务器 每执行一个写操作，就向 从服务器 发送相同的写命令，从服务器 接受并执行

#### 同步策略

主从刚刚连接的时候，进行全量同步

全同步结束后，进行增量同步