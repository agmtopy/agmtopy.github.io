---
title: redis基础知识
date: 2019-02-25 23:01:21
categories: 杂记
tags:
  - redis
  - 分布式锁
---

## redi分布式锁

### 实现原理
redis分布式锁实现的原理是，多个进程同时竞争同一个共享资源，拥有这个资源的线程就可以执行，其他进程等待执行线程释放。

### 问题
加锁：
1. set和get是两个命令，因此要用**set nx**
2. 不会自动过期，因此要用**px**,设定过期时间
3. 无法标记执行线程，因此要设置redis唯一值

```lua
 SET resource_name my_random_value NX PX 30000
```

解锁：
解锁需要判断线程持有的随机值和redis的值是否一样，因此要使用lua脚本

```lua
    if redis.call("get",KEYS[1]) == ARGV[1] then
        return redis.call("del",KEYS[1])
    else
        return 0
    end
```
在只是单机版redis的分布式锁，不适合分布式环境。当redis宕机后，执行线程不会被释放。当线程执行耗时任务时，可能出现redis释放锁，其他线程持有锁。
分布式redis锁可以采用Redission实现，底层是redis作者实现的Redlock原理。

### 基本数据结构

类型|特点
--|--
string|字符串
list|列表
set|不重复列表
hash|哈希
zset|有序集合

Redis中的hash是渐进式hash,不会在扩容时暂停，而是会重新创建一个新的hash来保存
