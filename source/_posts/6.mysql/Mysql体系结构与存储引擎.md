---
title: MySQL体系结构与存储引擎
date: 2021-03-15 21:54:51
categories: 数据库
tags:
  - MySQL
---

> 主要介绍MySql的整体体系结构和存储引擎


## 体系结构

概念上<B>数据库</B>是文件的集合,是按照一定的文件模型组织起来存放数据的结构;<B>数据库实列</B>是程序是一个用户进程,是用户对<B>数据库</B>进行操作的软件.

MySql主要由以下几部分组成
1. 连接池组件
2. 管理服务和工具组件
3. SQL接口组件
4. 查询分析器组件
5. 优化器组件
6. 缓冲组件
7. 插件式存储引擎(存储引擎是基于表的 ,而不是基于数据库的)
8. 物理文件

## 存储引擎

- 概述
存储引擎是与MySql物理文件进行交互的一种插件,存储引擎的底部是物理存储层,包括二进制日志文件,数据文件,错误文件,慢查询日志,undo/redo日志等

- select的过程

下图是一次select的过程
![select执行过程](https://s3.ax1x.com/2021/03/15/6r8Q7q.png)

从图中得知存储引擎位于是物理文件和程序数据之间


- InnoDB的体系架构

[![InnoDB架构](https://dev.mysql.com/doc/refman/8.0/en/images/innodb-architecture.png)](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)

1. 实列层
实列层主要包括'线程'和'内存'两个部分
a. 后台线程主要有
- Master Thread
主线程主要负责将缓冲池中的数据异步刷新到磁盘中,保证数据的一致性
- I/O Thread 
I/O Thread 主要是负责AIO(Async IO)请求的回调
- Purge Thread
Purge Thread的作用是事物在被提交后undolog就可能不在被使用了,因此需要Purge Thread线程来回收并分配的undo页
- Page Cleaner Thread
Page Cleaner Thread的作用是回收分配脏页

小结:Master Thread和I/O Thread一个负责异步刷新数据一个负责AIO的回调,Purge Thread和Page Cleaner Thread一个负责回收分配undolog一个负责回收分配脏页

b. 内存
- 缓冲池
InnoDB存储引擎是基于磁盘存储,其中的数据是按照页的方式进行管理.由于CPU和磁盘速度中的差异因此用缓冲池技术来提高数据库的整体性能
缓冲池是一块内存区域,在数据库进行读取页操作时,会首先将从内存中的数据放到缓冲池中,在下一次读取相同页时,会直接读取缓冲池中的页数据.
对于数据库中的页的修改,首先会修改缓冲池中的页,然后以一定的频率刷新到磁盘,这种机制被称为CheckPoint
缓冲池中的数据页类型分为:
1. 索引页
2. 数据页
3. undo页
4. 插入缓冲
5. 自适应哈希索引
6. InnoDB存储的锁信息
7. 数据字典

- LRU list,Freen lst,Flush list
数据库中的缓冲池是通过LRU算法来进行管理,但是InnoDB中的LRU算法是经过优化的,将新数据放到列表3/8处

- 重做日志缓冲
重做日志首先会放到缓冲区,然后按照一定的频率刷新到日志文件上,有三种情况会刷新重做日志:1.Master Thread每一秒都会将日志缓冲刷新到日志文件,2.每个事务提交时都会将重做日志刷新到文件,3.当重做日志缓冲池剩余空间小于1/2时,会将日志刷新到文件上

- 额外的内存池
额外的内存池作为缓冲池的备用,但缓冲池不够的时候,就会从额外的内存池中申请

- Checkpoint技术
Mysql中数据的持久性是通过Write Ahead Log策略来保证,也就是先写日志文件,在修改数据文件.当宕机导致数据丢失时,会通过重做日志来进行数据恢复.
如果重做日志太大会导致数据恢复的很慢,因此mysql使用CheckPoint的技术来保证,CheckPoint是将缓冲池中的内存页刷新到磁盘的动作




## Master Thread
Master Thread具有最高级别的线程优先级,内部有多个循环组成:主循环,后台循环,刷新循环,暂停循环.
Master Thread主要分为1s执行一次的操作:
1. 日志缓冲刷新到磁盘,即使这个事务还没提交
2. 合并插入缓冲
3. 最多刷新100个InnoDB的缓冲池中的脏页到磁盘
4. 如果当前没有用户活动,就切换到backgroud loop
10s执行一次的操作:
1. 刷新100个脏页到磁盘
2. 合并至多5个插入缓冲
3. 将日志缓冲刷新到磁盘
4. 删除无用的Undo页
5. 刷新100个或者10个脏页

## InnoDB特点
1. 插入缓冲(Insert Buffer)
2. 两次写(Double Write)
3. 自适应哈希索引(Adaptive Hash Index)
4. 异步IO(Async IO)
5. 刷新邻接页(Flush Neighbor Page)

## 插入缓冲
1. Insert Buffer
Insert Buffer是在插入数据时将辅助索引先放到Insert Buffer中,然后在按照一定的频率将Insert Buffer中的数据和辅助索引页的子节点进行合并,这样能够提高非聚集索引的插入性能
2. Change Buffer
Change Buffer作为Insert Buffer的升级,会将所有对非聚集索引的操作进行缓冲
3. Insert Buffer的内部结构
Insert Buffer的数据结构是B+树,在Mysql4.1之前每张表都有一颗B+树;在现在的版本中全局只有一颗B+树,负责对所有表的辅助索引进行Insert Buffer,默认为ibdata1
Insert Buffer的非叶子节点是由space(待插入记录表的表空间id),offer(偏移量)构成;叶子节点是由space,offer,metadata(操作辅助索引的元数据-顺序,标识,类型)
4. merge Insert Buffer的过程
在三种情况下,MySql会将Insert Buffer刷新到辅助索引文件中,并且虽然Insert Buffer是有序的,但是在刷新时,是随机选取Insert Buffer中的一个页进行刷新,做这个的目的是为了保证公平性而舍弃顺序性.以下的三种情况会进行Inser Buffer的刷新
1. 辅助索引页被读取到缓冲池中
2. Insert Buffer BitMap记录Insert Buffer可用空间不足时
3. Master Thread线程对Insert Buffer的刷新

## 两次写
两次写指的是当在直接写入某页时发生宕机,会有丢失数据的风险.通过向将脏页的数据转移到内存中的doublewrite buffer中,在将doublewrite中的数据同步写到共享表空间的物理磁盘文件上,第二次才将doublewrite buffer中的文件刷新到数据文件(.ibd)中,这样可以尽量保证数据文件的完整性

## 自适应哈希索引
自适应哈希索引指的是连续对热点数据的多次查询后会根据B+树索引生成对应的哈希索引,加快查询性能

## 异步IO
异步IO可以提高操作磁盘的效率,

## 刷新邻接页
刷新邻接页指的是对脏页所在区的页进行检测,发现是脏页一并进行刷新.