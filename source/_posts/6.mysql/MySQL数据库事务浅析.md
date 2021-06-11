---
title: MySQL数据库事务浅析
date: 2020-02-29 21:00:59
categories: 数据库
tags:
  - MySQL
---


# 数据库事务理解

数据库事务是一种本地的刚性事务，支持ACID

## **ACID**理论
名称|描述
--|--
原子性(Atomicity)|指的是在事务是最小的不可分割的执行单元，事务内的操作要么__都__成功，要么__都__失败
一致性(Consistency)|指的是在事务执行前后数据上的完整性保存一致，不会因为数据库事务的执行而改变数据在业务模型上的完整性
隔离性(isolation)|指的是每个事务都有单独的操作空间，不能在事务期间相互影响
持久性(Durability)|指的是事务完成后对数据库的改变是以落盘结束的

## 数据库的隔离级别

隔离级别|描述
--|--
Read Uncommitted(未提交读)|能够读取到其他事务尚未提交的数据
Read committed(读取已提交内容)|只能读取事务已提交的数据，
Repeatable Read(可重读)|指的是在同一个事务中任意时间节点读取到的内容一致
Serializable(串行化)|指的是事务是以串行化执行

## 事务异常
异常|描述
--|--
脏读（Dirty Read）|指的是读到了未提交的数据
不可重复读（Not Repeatable Read）|指的是在一个事务的过程中，前后读取到的数据行不一致
幻读|指的是在一个事务中读取到的数据行不一致

- 不可重复读指的是对数据的**删除**、**更新**操作
- 幻读指的是对数据的**inster**操作

## 数据库的隔离级别
- √为会发生，×为不会发生

隔离级别|脏读|不可重复读|幻读
--|--|--|--
read uncommitted（未提交读）|√|√|√
read committed（提交读）|x|√|√
repeatable read（可重复读）|x|x|√
serialization（可串行化）|x|x|x

## mysql实际操作

- 初始化语句
```sql
    CREATE TABLE `test` (
    `id` int(11) NOT NULL,
    `num` int(11) NOT NULL DEFAULT '0',
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
```

- mysql8之前版本
```sql
    SELECT @@tx_isolation;
```
- mysql8

```sql
    SELECT @@transaction_isolation;
```
- 查询结果
![查询结果](https://s2.ax1x.com/2020/03/01/3c8EEq.png)

mysql的默认事务是**REPEATABLE-READ**


### 脏读
- 设置隔离级别为**读未提交**(read uncommitted)

```sql
    SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

**A事务**
- 查询数据
```sql
    SELECT * FROM test;
```

- 结果

![查询结果](https://s2.ax1x.com/2020/03/01/3cOI29.png)

**B事务**
- 插入数据
```sql
    START TRANSACTION ;
    INSERT test VALUE(1,10);
```

**A事务**
- 查询数据

![查询结果](https://s2.ax1x.com/2020/03/01/3gxwH1.png)

**此时就读取到了未提交的数据**

**B事务**
- 回滚事务
```sql
    ROLLBACK;
```

**A事务**
- 查询数据
![查询结果](https://s2.ax1x.com/2020/03/01/32Kr5T.png)

### 不可重复读

- 结果
![查询结果](https://s2.ax1x.com/2020/03/01/32lyPf.png)

线程不能读取到其他线程尚未提交的数据


### 可重复读


![可重复读查询结果](https://s2.ax1x.com/2020/03/01/321bkt.png)

上图展示了在同一个事务中读取不管B事务是否提交，A事务读取的数据都是一样的。

- 幻读展示
![可重复读查询结果](https://s2.ax1x.com/2020/03/01/32tbkj.png)

可重复读的隔离级别虽然保证了在同一个事务中，前后读取数据一致，当时在插入时，由于是**表级锁**因此数据会等待之前的事务提交完成后再执行，因此出现异常。

> 参考文档
- [快速理解脏读，不可重复读，幻读](https://blog.csdn.net/Vincent2014Linux/article/details/89669762)
- [数据库的四种事物隔离级别](https://blog.csdn.net/qq_22115231/article/details/80767069)







