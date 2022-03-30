---
title: 《Java异步编程实战》笔记
date: 2020-12-12 16:45:36
categories: 杂记
tags:
  - 笔记
---

# 《Java异步编程实战》笔记

## 第一章 认识异步编程
基础概念和场景介绍。略...

## 第二章 显示使用线程和线程池实现异步编程
线程和线程池的使用做了个简介，重点讲了一下线程池的实现原理

- 线程池的实现原理<b>ThreadPoolExecutor</b>

```java
 final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            //如果当前还有任务
            while (task != null || (task = getTask()) != null) {
                w.lock();
                // 判断线程池状态
                if ((runStateAtLeast(ctl.get(), STOP) || (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP))) && !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        //执行任务
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
```

> 注意两个地方:1.Thread会被包装成worker对象用<b>HashSet</b>保存;2.Task是用<b>BlockingQueue</b>来进行保存

## 第三章 基于JDK中的Futrue实现异步编程

### FutureTask
- run()
- get()
- cancel()
> 注意一下FutureTask的state是用volatile进行修饰，结果outcome是用state的内存屏障技术来保证可见性的

### CompletableFuture
- runAsync()
- supplyAsync()
- thenRun()
- thenAccept()
- thenApply()
- whenComplete()
- thenCompose()
- thenCombine()
- allOf()
- anyOf()
- completeExeceptionally() 异常处理

> CompletableFuture默认是用ForkJoinPool,这里需要注意下

## Spring框架中的异步执行

### TaskExecutor
略...

### @Async

@EnableAsync是通过AsyncConfigurationSelector进行加载，

## 基于反应式编程实现异步编程

> 反应式编程是一种涉及数据流和变化传播的异步编程范式。这意味着可以通过所采用的编程语言轻松的表达静态或动态数据流。

反应式编程主要是强调对数据的处理过程是主动的，是以数据为核心参与对象的过程。传统的编程模型下是以业务流程为核心，通过业务流程将数据串联起来。

### Reactive Streams规范
>  Reactive Streams规范是提供一个使用非阻塞回压功能对异步流进行处理的标准，也就是定义反应式编程的规范

### 基于RxJava实现异步编程
==RxJava==是Reactive Extensions的java语言实现，具体的使用方法可参考官网


### 基于Reactor实现异步编程
==Reactor==是另外一个Java语言下的是Reactive实现，目前在webFlux中作为反应式框架进行使用


这里有一个软件发展的趋势就是传统的编程模型是基于cpu多核技术发展的还不是很好的时候提出来的，已经不太适用于现代多核cpu架构。因此基于现代cpu多核架构和传统软件架构面对的问题，提出了反应式编程模型。这种编程方式也许不是很便于人类理解，但是确拥有更高的资源利用率。如果未来出现一个像spring对于java那样对于反应式编程的框架，反应式编程理念应该会得到更好的发展。

## Web Servlet的异步非阻塞处理

在Servlet3.0中提供了异步处理能力，让请求线程和处理线程不在是1：1的关系了,这里就可以更好的利用服务器的性能。
这里有一个问题什么Servlet 3才提出的异步处理能力喃？
这是因为servlet是基http1.1的方案，http1.1开始支持http可以保持长连接的形式，hhtp1.1方案是1999年开始
在Servlet3.1中提供了非阻塞I/O的处理方式:Web容器中的非阻塞请求处理有助于增加Web容器可同时处理请求的数量，允许我们在ServletInputStream上通过函数setReadListener注册一个监听器，该监听器在发现内核中有数据时才会进行回调处理函数。

### spring MVC的异步处理能力

>Spring MVC是围绕前端控制器模式设计，由中央处理器Servlet DispatcheaServlet作为请求处理进行路由分派，实际请求处理工作由可配置处理类进行执行。在Spring MVC中通过调用request.startAsync()将ServletRequest设置为异步模式，这样可以让Servlet在推出的同时，让响应保持打开状态。

## Spring web Flux
webFlux与spring mvc的对比
![webFlux vs spring mvc](https://docs.spring.io/spring-framework/docs/current/reference/html/images/spring-mvc-and-webflux-venn.png)









## 参考资料
[infoworld关于servlet 3.0的介绍](https://www.infoworld.com/article/2077995/java-concurrency-asynchronous-processing-support-in-servlet-3-0.html?page=1)
[spring官网关于webFlux的文档](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)


