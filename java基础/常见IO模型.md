https://www.cnblogs.com/javaguide/p/io.html

## BIO(Blocking-I/O)

同步阻塞模型

同步阻塞 IO 模型中，应用程序发起 read 调用后，会一直阻塞，直到在内核把数据拷贝到用户空间。

<img src="C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210312173444113.png" alt="image-20210312173444113" style="zoom:50%;" />



## NIO(Non-Blocking I/O/NEW I/O)

Java 中的 NIO 可以看作是 **I/O 多路复用模型**



<img src="C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210312174930222.png" alt="image-20210312174930222" style="zoom:50%;" />

IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间->用户空间）还是阻塞的。

> 目前支持 IO 多路复用的系统调用，有 select，epoll 等等。select 系统调用，是目前几乎在所有的操作系统上都有支持
>
> - **select 调用** ：内核提供的系统调用，它支持一次查询多个系统调用的可用状态。几乎所有的操作系统都支持。
> - **epoll 调用** ：linux 2.6 内核，属于 select 调用的增强版本，优化了 IO 的执行效率。

#### 选择器

Java 中的 NIO ，有一个非常重要的**选择器 ( Selector )** 的概念，也可以被称为 **多路复用器**。通过它，只需要一个线程便可以管理多个客户端连接。当客户端数据到了之后，才会为其服务。

<img src="C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210312175456148.png" alt="image-20210312175456148" style="zoom:80%;" />



## AIO(Asynchronous I/O)

异步NIO

异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

<img src="C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210312175645557.png" alt="image-20210312175645557" style="zoom:50%;" />

## 总结

<img src="C:\Users\29189\AppData\Roaming\Typora\typora-user-images\image-20210312175815864.png" alt="image-20210312175815864" style="zoom:80%;" />





