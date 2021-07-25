---
title: Arthas初探--安装初步适用
date: 2019-07-29 22:40:08
categories: 杂记
tags:
  - linux
  - arthas
---

# Arthas初探--安装初步适用
由于在项目中遇到一种情况，某段代码在进行单元测试和在tomcat容器中运行的性能相差数百倍，因此需要分析在不同环境下某个方法执行的具体时间，从而确定问题。Arthas可以做到无侵入的监控应用远行情况。

## 安装

githup项目地址：**https://github.com/alibaba/arthas**
文档地址：**https://alibaba.github.io/arthas/**

安装
```bash
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar
```
linux下直接执行，window下载文件后执行

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190630153613103.png)
执行完成后，显示当前path中指定的JDK中正在运行的java进程
输入相应序号,进入sh命令，表示已连接成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019063015385964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMjk0Mzc=,size_16,color_FFFFFF,t_70)

 [^1]: [mermaid语法说明](https://mermaidjs.github.io/)

## 初步使用
Arthas命令初步使用，大概分为5类：
### 基础命令
- help——查看命令帮助信息
- cat——打印文件内容，和linux里的cat命令类似
- pwd——返回当前的工作目录，和linux命令类似
- cls——清空当前屏幕区域
- session——查看当前会话的信息
- reset——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
- version——输出当前目标 Java 进程所加载的 Arthas 版本号
- history——打印命令历史
- quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
- shutdown——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
- keymap——Arthas快捷键列表及自定义快捷键
### jvm相关
- dashboard——当前系统的实时数据面板
- thread——查看当前 JVM 的线程堆栈信息
- jvm——查看当前 JVM 的信息
- sysprop——查看和修改JVM的系统属性
- sysenv——查看JVM的环境变量
- getstatic——查看类的静态属性
- New! ognl——执行ognl表达式
- New! mbean——查看 Mbean 的信息

### class/classloader相关
- sc——查看JVM已加载的类信息
- sm——查看已加载类的方法信息
- jad——反编译指定已加载类的源码
- mc——内存编绎器，内存编绎.java文件为.class文件
- redefine——加载外部的.class文件，redefine到JVM里
- dump——dump 已加载类的 byte code 到特定目录
- classloader——查看classloader的继承树，urls，类加载信息，使用classloader去getResource

### monitor/watch/trace相关
请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 shutdown 或将增强过的类执行 reset 命令。
- monitor——方法执行监控
- watch——方法执行数据观测
- trace——方法内部调用路径，并输出方法路径上的每个节点上耗时
- stack——输出当前方法被调用的调用路径
- tt——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

### options
- options——查看或设置Arthas全局开关

## 使用实列
- trace 分析每个方法的具体执行时间
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190630154629460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMjk0Mzc=,size_16,color_FFFFFF,t_70)
通过图示表明调用MongoTemplate.executeFindMultiInternal()方法时，最耗时的方法是在doWith()方法，总共执行10000次，耗时==252.3064ms==,最少一次调用耗时==0.0132ms==，最大一次耗时==38.4329ms==，分析原因还是在于数据量太大，MongoTemplate通过循环遍历出结果在进行序列化。


- jad 反编译代码工具
```bash
 jad com.sankuai.inf.leaf.common.ZeroIDGen
```

![aVeudO.png](https://s1.ax1x.com/2020/07/28/aVeudO.png)

- watch 查看输入参数与输出参数
```bash
    watch com.sankuai.inf.leaf.server.service.SegmentService getId '{params, target, returnObj}' -x 2
```

> **params**表示入参，**target**表示当前的类，**returnObj**表示返回值

![aVmDAO.png](https://s1.ax1x.com/2020/07/28/aVmDAO.png)

- stack 查看被调用的路径(向上)

```bash
 stack com.sankuai.inf.leaf.server.service.SegmentService getId
```

![aVKIUK.png](https://s1.ax1x.com/2020/07/28/aVKIUK.png)

- sc 查看JVM已加载的类信息

```bash
sc -d com.sankuai.inf.leaf.server.service.SegmentService getId
```

![aVMArn.png](https://s1.ax1x.com/2020/07/28/aVMArn.png)


- thread 分析死锁
```bash
 thread b
```

![JHNalT.png](https://s1.ax1x.com/2020/04/29/JHNalT.png)

![JHUuNR.png](https://s1.ax1x.com/2020/04/30/JHUuNR.png)

可以看出当前线程正在等待**ReentrantLock$NonfairSync@118f1fb4**，而持有这个对象的线程又在等待当前线程释放，从而形成死锁!

- thread 分析CPU占用

![JH0HQx.png](https://s1.ax1x.com/2020/04/30/JH0HQx.png)


## 总结
 先放一张官方的总结大图
 [![JHBSfA.png](https://s1.ax1x.com/2020/04/30/JHBSfA.png)](https://imgchr.com/i/JHBSfA)


总结：Arthas是一个很优秀的java诊断工具，无论是安装还是使用都很简洁，并且使用文档全面、清晰明了，值得好好研究一番。

