---
title: Guava的使用
date: 2020-05-20 00:54:09
categories: 杂记
tags:
  - guava
---

# Guava的使用

## 线程池的使用

> ListenableFuture是基于装饰器模式实现的

- 示例
```java
String nameFormat = "thread_factory_%d";
        ThreadFactory shardingThreadFactory = ShardingThreadFactoryBuilder.build(nameFormat);
        ExecutorService delegate = Executors.newCachedThreadPool(shardingThreadFactory);
        //得到装饰器的线程池
        ListeningExecutorService executorService = MoreExecutors.listeningDecorator(delegate);
        ListenableFuture<String> future = executorService.submit(() ->{
            TimeUnit.SECONDS.sleep(3);
            return "线程:"+Thread.currentThread().getName() +",休眠3s";
        });
        //注册回调函数
        future.addListener(()->{
                System.out.println("线程:"+Thread.currentThread().getName()+"执行回调函数得到返回值为:" + future.get());
        },Executors.newSingleThreadExecutor(shardingThreadFactory));

```