---
title: 深入剖析Kubernetes之容器化的基础
date: 2021-10-28 14:08:23
categories: 容器化
tags:
  - docker
---


# 深入剖析Kubernetes之容器化的基础

## 容器化的基础之进程管理
<B>容器化的基础</B>是依赖于Linux底层提供的两种能力分别是<B>cgroups</B>,<B>Namespace</B>

### cgroups
> cgroups 的全称是control groups,是Linux内核提供的一种可以限制单个进程或者多个进程所使用资源的机制
cgroups可以管理的资源有:
- <B>cpu子系统</B>
    主要限制进程的cpu使用率
- <B>cpuacct子系统</B>
    可以统计cgroups中的进程的cpu使用报告
- <B>cpuset 子系统</B>
    可以为cgroups中的进程分配单独的cpu节点或者内存节点
- <B>memory子系统</B>
    可以限制进程的 </B>
    可以限制进程的块设备 io
- <B>devices子系统</B>
    可以控制进程能够访问某些设备
- <B>net_cls子系统</B>
    可以标记cgroups中进程的网络数据包，然后可以使用tc模块(traffic control)对数据包进行控制
- <B>freezer子系统</B>
    可以挂起或者恢复cgroups中的进程
- <B>ns子系统</B>
    可以使不同cgroups下面的进程使用不同的namespace

<B>cgroups</B>是通过与内核的其他模块来完成<B>进程维度</B>对<B>资源</B>的管理

### Namespace
<B>namespace</B>是对内核资源进行分组管理的一个特性,用来修改进程间的可见性

namespace创建进程时会使用<B>CLONE_NEWPID</B>这个参数来创建一个新的进程空间


### 虚拟机与Docker的区别

虚拟机与Docker的区别在于虚拟机会采用硬件虚拟化功能,模拟出操作系统底层的设备;docker只是在宿主机上做了进程隔离和资源隔离

## 容器化的基础之隔离和限制

![容器化与虚拟机之间的对比](https://i.loli.net/2021/10/28/HNTkSg5vwd7jymf.jpg)


通过<B>namespace</B>来完成进程之间的隔离,通过<B>cgroups</B>来完成对资源的限制

<B>cgroups</B>是通过配置在<B>/sys/fs/cgroup</B>文件夹下会有

- 资源分组
![cgroup资源分组](https://i.loli.net/2021/10/28/gUHWXSlaGd7Lpn8.jpg)
这些限制项来进行处理

- 控制组
![cgroup详细控制](https://i.loli.net/2021/10/28/PFv8caUqsLiYBpo.jpg)

通过详细信息可以看到这个目录下有不同的配置文件<B>cgroup.procs</B>,内容如下展示

[![cgroup.procs内容](https://z3.ax1x.com/2021/10/29/5jdCIx.jpg)
可以看到这里记录的就是pid列表













## 参考资料

[Linux资源管理之cgroups简介](https://tech.meituan.com/2015/03/31/cgroups.html)
[什么是命名空间和cgroup,以及它们是如何工作的？](https://www.nginx.com/blog/what-are-namespaces-cgroups-how-do-they-work/)