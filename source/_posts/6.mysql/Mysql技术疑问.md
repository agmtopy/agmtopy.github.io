---
title: MySQL技术疑问
date: 2021-03-10 18:12:31
categories: 数据库
tags:
  - MySQL
---
> 记录<<MySQL技术内幕-InnoDB存储引擎>>阅读中存在的疑问点

- P77 日志文件
> 如果使用RC隔离级别会出现类似数据丢失更新的现象,从而出现主从数据库上的数据不一致

bin_log的记录格式为'STATENEBT'是,会出现这样的情况为什么喃?

原因:
> 1. bin_log的记录顺序是commit的顺序,而不是执行顺序
2. RC中SQL的执行是立即执行的
这样如果bin_log记录格式如果是sql,就会出现丢失数据的场景

-------------

- P268 锁
> InnoDB对于Insert的操作会检查插入记录的下一条记录是否被锁定,如果锁定则不允许操作

- P269 解决幻读
> 在RR模式下通过Next-Key Locking机制来解决幻读,既然MVVC是读取快照,为什么还需要Next-Key Locking机制来解决幻读问题

InnoDB实际上是把读取拆分成两种类型
- 快照读
通过读取undo log,并且对比版本号得到事务开始时的数据,这种主要是select 操作,对数据加s锁
- 当前读
当前读指的是通过Next-Key Locking 机制将对数据的读取操作序列化,解决幻读问题


-----------------

- checkpoint机制
> checkpoint机制指的是将内存中的数据刷新到磁盘上的机制,内存中的数据包括<B>数据页</B>、<B>索引页</B>、<B>undo页</B>、<B>插入缓冲 insert buffer</B>、<B>redo log buffer</B>

MySQL中内存页淘汰算法比较特殊，采用的是基于LRU-最近最少的算法改进后(3/4插入新值)进行淘汰的，防止一个表的大量更新影响到其他热点数据


<B>show engine innodb status</B>查看innodb engine的状态


---------------------------------------------

- InnoDB存储引擎特点

1. 插入缓冲(insert buffer)
> insert buffer是对非聚集索引页的插入,修改缓冲页就直接认为成功后续线程在专门将page刷新到磁盘中；change buffer是对insert buffer的升级

2. 两次写(Double Write)
   double write产生的根本原因是由于由于MySql的最小写入单位为16K,而文件系统能保证原子写入的最小单位是4k，这样就造成MySql一次性向文件系统写入16k的数据时可能会发生中断，会污染数据页，通过先写中间层的方式来解决这个问题

3. 自适应哈希索引(Adaptive Hash Index)
4. 异步IO(Async IO)
   MySQL中大量使用到异步IO

5. 刷新邻接页(Flush Neighbor Page)


--------------------------

- bin log的日志格式

<B>STATENEBT</B>模式，记录的是标准的逻辑SQL语句
<B>ROW</B>模式，记录的是每一行的数据
<B>MIXED</B>模式混合模式7



--------------------------

意向锁是表级别的锁，分为<B>意向写锁</B>和<B>意向读锁</B>


Innodb默认支持一致性非锁定读(指的是可以不用等待写锁释放,通过undo log来来获取数据);RC模式与RR模式读取的数据不一样


行锁有三种类型：
1. 行上面的X锁
2. GAP间隙锁
3. Next-Lock:混合模式

默认的存储引擎RR是通过Next-Lock来实现避免幻读的


## 参考资料
- [知乎MVCC的解答](https://www.zhihu.com/question/334408495)




