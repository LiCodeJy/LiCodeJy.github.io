## Redis Sentinel 哨兵模式

哨兵模式能够后台监控主机是否故障，如果故障根据投票数**自动将从库切换为主库**

Redis提供了哨兵的命令，作为一个独立的进程监控Redis服务器，哨兵通过发送命令等待服务器响应，从而监控运行的多个Redis实例

#### 哨兵的作用

- 监控：通过发送命令，监控redis服务器的状态
- 故障转移：如果发现master宕机，会自动将slave切换为master，并通过发布订阅者模式通知其他从机，让它们切换主机

### 单个哨兵（实际不使用，只作为理解）

<img src="D:%5CTypora%5Cworkspace%5Cimg%5C11320039-57a77ca2757d0924.png" alt="img" style="zoom:80%;" align='left'/>

只有一个哨兵，如果哨兵出现问题，就会导致哨兵模式失效，所以有了多哨兵模式

### 多哨兵模式

<img src="D:%5CTypora%5Cworkspace%5Cimg%5C11320039-3f40b17c0412116c.png" alt="img" style="zoom:80%;" align='left'/>

#### 哨兵之间的交互

- 多个哨兵之间通过发布订阅者模式交互，类似广播
- 每个哨兵都会订阅（SUBSCRIBE）`__sentinel__:hello` 这个频道，接收其他哨兵信息以及主节点信息
- 每隔两秒也会通过这个频道发送（PUBLISH）自己的 ip、端口、runid 以及 主节点 名称、ip、端口 等信息

通过 SUB/PUB 的交互，哨兵之间能够实现自动发现，也能及时更新自己保存的信息

#### 哨兵的故障检测

Redis 以类似心跳检测的 PING 命令对节点进行健康检测（默认为1s发送一次）

- SDOWN 主观宕机（Subjective Down）: 一个哨兵自己认为主节点宕机

- ODWON客观宕机（Objective Down）: 不仅一个哨兵自己认为主节点宕机，并且和其他哨兵沟通（向其他哨兵发送Sentinel is-master-down-by-addr 命令）后，**一定数量**的哨兵认为主节点宕机

  - 一定数量 是指哨兵配置中的 Quorum

  - 哨兵配置：sentinel monitor <master-name> <maseter-host> <master-port> <quotum>

    ​		示例：sentinel monitor  mymaster 			127.0.0.1 				6379 						2

ODOWN 状态才会触发 failover 故障转移

#### 故障转移

- 哨兵之间先选举出一个 Sentinel Leader
- Sentinel Leader 选出一个从机作为下一任主机
- Sentinel Leader会向它发送`slaveof NO ONE` 命令，这个从机就变为了主机
- 遍历原主节点的从节点，向这些从节点发送 slaveof <ip> <port> 命令 

#### 总结

- 多个哨兵发现并确认 master有问题
- 哨兵中选出一个 leader
- 选出一个 slave 作为新的 master
- 通知其余 slave 修改自己的 master
- 通知客户端主从变化
- 老 master 复活则成为新 master 的 slave