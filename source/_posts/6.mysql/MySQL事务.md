---
title: MySQL事务
date: 2021-03-27 15:09:33
categories: 数据库
tags:
  - Mysql
---

## 事务的定义
事务是数据库与文件系统最重要的差异之一,数据库通过事务保证了数据的ACID特性,分别是原子性/一致性/隔离性/持久性

## 事务的分类
事务可以划分为:
1.扁平事务以及带有保存点的扁平事务
2.链事务
3.嵌套事务
4.分布式事务

## 事务的实现
InnoDB通过日志系统来实现事务,redo log可以保证事务的原子性/一致性/持久性,undo log可以保证事务的原子性和持久性

### redo log的作用
redo log是重做日志,通过每次事务提交前先修改该事务要修改的页,在提交过程中如果中断,通过redo log就可以继续提交事务

### undo log的作用
undo log是回滚日志,通过记录每个事务开始时的数据,在回滚发生中断时,可以根据undo log 继续进行事务回滚

## redo log
redo log由两部分组成'redo log buffer'和'redo log file',在进行事务connit之前都会将事务的所有日志写入日志文件中进行持久化
InnoDB提供一个参数'innodb_flush_log_at_trx_commit'来控制redo log的刷盘策略,0:由master thread进行控制 1:每次提交后进行同步刷盘 2:每次提交后只是将数据提交给文件系统,不进行主动刷盘操作


## undo log
undo log是回滚日志,是逻辑日志并不是redo log的物理日志,只是在逻辑上保证了数据回滚到原始状态.例如用户执行一个insert操作,在undo log里面就需要一个delete操作

## MVVC

多版本控制,InnoDB会在行记录上默认增加两个隐藏列来作为MVVC实现的基础,分别是DB_TRX_ID-最新一次事务提交版本号/DB_ROLL_PTR-删除事务的版本号
数据行和undo log组成了不同版本之间的数据链,通过对比版本号和undo log日志将所需要的数据还原出来







