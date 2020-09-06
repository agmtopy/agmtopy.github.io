---
title: Linux环境排查问题的思路
date: 2020-04-29 00:28:05
categories: 杂记
tags:
  - linux
  - arthas
---

# Linux环境排查问题的工具和思路

将工作中使用到的在Linux下进行排查问题的工具和思路进行一个总结，主要从**CPU**、**内存**、**I/O**、**网络**、**JVM**这五个方面来进行。

## **CPU**部分

### CPU的性能指标

- 图示
[![JoptbV.png](https://s1.ax1x.com/2020/04/29/JoptbV.png)](https://imgchr.com/i/JoptbV)

### CPU的性能分析工具

性能指标|性能工具|描述
--|--|--
系统CPU使用率|top、vmstat、mpstat、sar、/proc/stat|top、vmstat、mpstat作为实时的监控数据;sar可记录历史;/proc/stat
进程CPU使用率|top、ps、pidstat、htop、atop|top和ps可以按照cpu使用率进行查看;pidstat展示实际运行的进程;htop和atop可以以不同颜色进行展示
平均负载|uptime、top、/proc/loadavg|uptime简洁、top全面、/proc/loadavg提供监控数据
系统上下文切换|vmstat|提供系统级的上下文切换次数
进程上下文切换|pidstat|提供进程级的上下文切换次数
软中断|top、/proc/softirqs、mpstat|
硬中断|vmstat、/proc/interrupts|vmstat提供总的中断次数、/proc/interrupts提供在每个cpu上的中断次数
网络|dstat、sar、tcpdump|dstat和sar提供总的网络接收情况和发送情况、tcpdump是提供动态抓取正在进行的网络通信
I/O|dstat、sar|都是提供整体的I/O情况

### CPU的分析步骤
> **top** -> **vmstat** -> **pidstat** ->进程分析工具

总的思路就是根据**top**获取到系统当前的一个状态，根据当前的状态在进行对应的分析

## 内存

### 内存分析工具

性能指标|性能工具|描述
--|--|--
系统已用、可用、剩余内存|free、vmstat、/proc/meminfo|获取内存信息
进程虚拟内存、常驻内存、共享内存|ps、top
进程内存分布|pmap|

### 内存分析流程

> **free** -> **vmstat和sar** -> 确定内存问题





### 实例

- htop
[![Jo9FaT.png](https://s1.ax1x.com/2020/04/29/Jo9FaT.png)](https://imgchr.com/i/Jo9FaT)