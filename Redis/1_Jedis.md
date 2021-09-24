## Jedis

使用Java来操作Redis的中间件

pom依赖

```XML
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.6.3</version>
</dependency>
```

测试

```java
public class TestJedis {
    public static void main(String[] args) {
        HostAndPort hostAndPort = new HostAndPort("127.0.0.1", 6379);
        //通过主机和端口号连接
        Jedis jedis = new Jedis(hostAndPort);
        //返回一个事务对象
        Transaction multi = jedis.multi();
        multi.set("a","2");
        multi.set("b","1");
        System.out.println(multi.exec());
        System.out.println(jedis.get("a"));
        //断开连接
        jedis.close();
    }
}
```

Jedis中方法名和Redis中的命令相同，不需要特别测试了