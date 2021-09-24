## Redis持久化

### RBD（Redis DataBase）

执行持久化操作时，redis  fork 一个子进程，先将数据集快照写入一个临时文件，写入成功后替换掉之前的文件

rbd保存的文件是dump.rdb

**产生 rdb 文件触发机制：**

- conf 文件中SNAPSHOTTING配置的 save 规则满足

  ```bash
  save 900 1
  save 300 10
  save 60 10000
  ```

- 执行 flushall 命令

- 退出 redis

**使用rdb文件恢复数据**

- 将 dump.rdb 放在启动目录下

- redis 启动会自动检查 dump.rdb，恢复其中的数据

- 查看启动目录

  ```bash
  127.0.0.1:6379> config get dir
  1) "dir"
  2) "D:\\Redis-x64-3.0.504"
  ```

优点：性能最大化（由子进程完成持久化操作）、易恢复

缺点：数据完整性不高

### AOF（Append Only File）

以日志的形式将所有的写操作记录下来（不记录读操作），只许追加文件但不允许改写文件，恢复时将所有的命令都重新执行一遍

aof保存的文件是appendonly.aof

Redis提供了三种同步策略

- 每秒同步
- 每修改同步
- 不同步

在 redis.conf 中配置

````bash
appendonly yes	#开启aof模式

# appendfsync always	#每修改同步
appendfsync everysec	#每秒同步
# appendfsync no		#不同步
````

持久化文件有问题，可以用 redis-check-aof来修复

优点：

- 每次修改都同步，数据完整性很高
- 每秒同步，可能会丢失1秒数据

缺点：

- aof 远大于 rdb 文件，修复速度也比 rdb 慢
- 效率比 rdb 低

