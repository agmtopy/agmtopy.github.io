---
title: MySQL技术疑问
date: 2021-03-20 18:12:31
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


## 参考资料
- [知乎MVCC的解答](https://www.zhihu.com/question/334408495)




