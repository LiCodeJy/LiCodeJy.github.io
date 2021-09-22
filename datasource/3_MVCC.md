## MVCC

**看这个** [数据库基础（四）Innodb MVCC实现原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/52977862)

multiversion concurrency control多版本并发控制，实现了非阻塞的读操作，写操作也只锁定必要的行

MVCC的实现依赖于 **隐藏字段、Read View、undo log**

### 隐藏字段

在内部，`InnoDB` 存储引擎为每行数据添加了三个隐藏字段：

- `DB_TRX_ID（6字节）`：表示最后一次插入或更新该行的事务 id。此外 delete 操作在内部被视为更新，只不过会在记录头 Record header 中的 deleted_flag 字段将其标记为已删除
- `DB_ROLL_PTR（7字节）` 回滚指针，指向该行的 `undo log` 。如果该行未被更新，则为空
- `DB_ROW_ID（6字节）`：如果没有设置主键且该表没有唯一非空索引时，`InnoDB` 会使用该 id 来生成聚簇索引

### ReadView

主要用于可见性判断

在innodb 中每个SQL语句执行前都会得到一个read_view。副本主要保存了当前数据库系统中正处于活跃（没有commit）的事务的ID号，其实简单的说这个副本中保存的是系统中当前不应该被本事务看到的其他事务id列表。

#### Read view 的几个重要属性

**trx_ids:** 当前系统活跃(未提交)事务版本号集合。

**low_limit_id:** 创建当前read view 时“当前系统最大**事务版本号**+1”。

**up_limit_id:** 创建当前read view 时“系统正处于**活跃事务**最小版本号”

**creator_trx_id:** 创建当前read view的事务版本号；

#### Read view 匹配条件

**（1）数据事务ID <up_limit_id 则显示**

如果数据事务ID小于read view中的最小活跃事务ID，则可以肯定该数据是在当前事务启之前就已经存在了的,所以可以显示。

**（2）数据事务ID>=low_limit_id 则不显示**

如果数据事务ID大于read view 中的当前系统的最大事务ID，则说明该数据是在当前read view 创建之后才产生的，所以数据不予显示。

**（3） up_limit_id <=**数据事务ID<**low_limit_id 则与活跃事务集合**trx_ids**里匹配**

如果数据的事务ID大于最小的活跃事务ID,同时又小于等于系统最大的事务ID，这种情况就说明这个数据有可能是在当前事务开始的时候还没有提交的。

所以这时候我们需要把数据的事务ID与当前read view 中的活跃事务集合trx_ids 匹配:

**情况1:** 如果事务ID不存在于trx_ids 集合（则说明read view产生的时候事务已经commit了），这种情况数据则可以显示。

**情况2：** 如果事务ID存在trx_ids则说明read view产生的时候数据还没有提交，但是如果数据的事务ID等于creator_trx_id ，那么说明这个数据就是当前事务自己生成的，自己生成的数据自己当然能看见，所以这种情况下此数据也是可以显示的。

**情况3：** 如果事务ID既存在trx_ids而且又不等于creator_trx_id那就说明read view产生的时候数据还没有提交，又不是自己生成的，所以这种情况下此数据不能显示。

**（4）不满足read view条件时候，从undo log里面获取数据**

当数据的事务ID不满足read view条件时候，从undo log里面获取数据的历史版本，然后数据历史版本事务号回头再来和read view 条件匹配 ，直到找到一条满足条件的历史数据，或者找不到则返回空结果；

#### 模拟MVCC实现流程

下面我们通过开启两个同时进行的事务来模拟MVCC的工作流程。

**（1）创建user_info表，插入一条初始化数据**

![img](https://pic3.zhimg.com/80/v2-2d342b1506470aeb3d1a37194058d07e_720w.jpg)



**（2）事务A和事务B同时对user_info进行修改和查询操作**

事务A：update user_info set name =”李四”

事务B：select * fom user_info where id=1

<img src="https://pic3.zhimg.com/80/v2-f52d8559ba58256df25735e761fd7ae2_720w.jpg" alt="img" style="zoom:150%;" />

1. 事务A先开启，DB_TRX_ID为102

2. 事务B开启，DB_TRX_ID为103

3. 事务A执行update操作，历史记录存入undo_log中

4. 事务B进行查询，获取read_view，其中：

   low_limit_id=当前事务+1即104

   up_limit_id=当前未提交的事务的最小版本，即102

   trx_ids=当前未提交的事务列表，即102，103

   creator_trx_id=当前事务id即103

5. 事务B对read view进行匹配，发现up_limit_id<=102<low_limit_id,且在trx_ids中，则通过undo log获取数据

6. 返回的是事务A修改前的数据

### **各种事务隔离级别下的Read view 工作方式**

RC(read commit) 级别下同一个事务里面的每一次查询都会获得一个新的read view副本。这样就可能造成同一个事务里前后读取数据可能不一致的问题（重复读）

![img](https://pic3.zhimg.com/80/v2-0c77f30980dc7e45f5aaac8a574e8672_720w.jpg)

RR(重复读)级别下的一个事务里只会获取一次read view副本，从而保证每次查询的数据都是一样的。

![img](https://pic1.zhimg.com/80/v2-82eeabba61c97def5d19aeb3cb77182c_720w.jpg)



READ_UNCOMMITTED 级别的事务不会获取read view 副本。

### 快照读和当前读

**快照读**

快照读是指读取数据时不是读取最新版本的数据，而是基于历史版本读取的一个快照信息（mysql读取undo log历史版本) ，快照读可以使普通的SELECT 读取数据时不用对表数据进行加锁，从而解决了因为对数据库表的加锁而导致的两个如下问题

1、解决了因加锁导致的修改数据时无法对数据读取问题;

2、解决了因加锁导致读取数据时无法对数据进行修改的问题;

因为快照读只获取一次read view，就算插入数据，也是返回undo log中的值，所以不会产生幻读

**当前读**

当前读是读取的数据库最新的数据，当前读和快照读不同，因为要读取最新的数据而且要保证事务的隔离性，所以当前读是需要对数据进行加锁的（Update delete insert select ....lock in share mode select for update 为当前读）

在当前读下，读取的都是最新的数据，如果其它事务有插入新的记录，并且刚好在当前事务查询范围内，就会产生幻读！

InnoDB 使用 **Next-key Lock** 来防止这种情况。当执行当前读时，会锁定读取到的记录的同时，锁定它们的间隙，防止其它事务在查询范围内插入数据。只要我不让你插入，就不会发生幻读。

> Next-Key Lock是行锁和间隙锁的组合，当InnoDB扫描索引记录的时候，会首先对索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。 加上间隙锁之后，其他事务就不能在这个间隙修改或者插入记录。