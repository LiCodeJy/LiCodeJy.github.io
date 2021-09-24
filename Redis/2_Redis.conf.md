## Redis的配置

1. 容量单位，不区分大小写

   <img src="D:%5CTypora%5Cworkspace%5Cimg%5Cimage-20210922203345783.png" alt="image-20210922203345783" style="zoom:80%;" align='left'/>

2. 可以包含其他配置文件

   <img src="D:%5CTypora%5Cworkspace%5Cimg%5Cimage-20210922203301805.png" alt="image-20210922203301805" style="zoom:80%;" align='left'/>

3. 网络配置

   ```bash
   bind 127.0.0.1	#绑定的ip
   protected-mode yes	#受保护模式
   port 6379	#端口号
   ```

4. 通用配置GENERAL

   ```bash
   daemonize yes	#是否以守护进程运行，默认为no，手动开启
   
   pidfile /var/run/redis_6379.pid	#如果以守护进程运行，就需要指定一个pid文件
   
   # Specify the server verbosity level.
   # This can be one of:
   # debug (a lot of information, useful for development/testing)
   # verbose (many rarely useful info, but not a mess like the debug level)
   # notice (moderately verbose, what you want in production probably)
   # warning (only very important / critical messages are logged)
   loglevel notice	#日志级别
   logfile ""		#生成的日志文件名
   
   database 16		#默认的数据库数量
   ```

5. 快照配置

   与持久化有关，在规定的时间内执行了多少次操作，则会持久化到 .rdb 和 .aof 文件

   ```bash
   # save <seconds> <changes>
   save 900 1		# 如果900s内至少有一个key进行了修改，则持久化
   save 300 10		# 60s内有10个key修改，持久化
   save 60 10000	# ...
   
   stop-writes-on-bgsave-error yes	#持久化发生错误是否继续工作
   
   rdbcompression yes	#是否压缩rdb文件
   rdbchecksum	yes		#是否校验rdb文件
   dir ./	#rdb文件保存的目录
   ```
   
6. REPLICATION主从复制配置

   为从机配置主机

   ```bash
   slaveof <masterip> <masterport>
   #<masterip>   主机ip
   #<masterport> 主机端口号
   ```

7. SECURITY安全设置

   ```bash
   requirepass # 密码，默认为空
   ```
   
8. APPEND ONLY模式 aof配置

   ```bash
   appendonly no	# 默认不开启aof模式，默认使用rdb方式持久化
   
   appendfilename "appendonly.aof"	# aof持久化的文件名
   
   appendfsync everysec	# 每秒执行一次sync同步，可能会丢失1s的数据
   appendfsync always		# 每次修改都同步
   appendfsync no 			# 不执行sync
   ```

   

 