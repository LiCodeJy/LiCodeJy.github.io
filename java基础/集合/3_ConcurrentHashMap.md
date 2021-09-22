# ConcurrentHashMap

## 存储结构

### JDK1.7

<img src="C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210316185800132.png" alt="image-20210316185800132" style="zoom:80%;" />

分为很多个segment段组合，分别对每个segment加锁

每个segment相当于一个小HashMap的结构，由数组+链表组成

segment中的数组和链表可扩展，但segment的个数一旦确定，不可扩展，默认为16个，相当于concurrentHashMap默认支持16线程并发

- 默认参数

```java
    /**
     * 默认初始化容量
     */
    static final int DEFAULT_INITIAL_CAPACITY = 16;

    /**
     * 默认负载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 默认并发级别
     */
    static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

### JDK1.8

存储结构

![image-20210316210342320](C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210316210342320.png)

1.8中使用Node数组+链表/红黑树

并发控制使用 synchronized 和 CAS 来操作

synchronized 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发

ConcurrentHashMap的初始化是通过自旋和CAS完成的

#### put

1.根据key算出hashCode

2.判断是否需要初始化

3.如果当前key定位到的Node为空，可以直接CAS写入，失败则自旋保证成功

4.如果当前位置的hashCode == MOVED == -1则需要扩容

5.如果不需要扩容，则用synchronized锁写入数据

6.如果数量大于*TREEIFY_THRESHOLD*转为红黑树

> CAS
>
> 