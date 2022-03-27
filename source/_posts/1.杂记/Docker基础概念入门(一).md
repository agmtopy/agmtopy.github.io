---
title: Docker基础概念入门(一)
date: 2020-09-15 22:44:43
categories: Docker
tags:
  - Docker
---

## Docker实战

### 基础命令
- 容器使用

功能|命令|参数
--|--|--
获取镜像|docker pull imageName|
启动镜像(create&start)|docker run imageName|-i:交互式 -t:终端 -d:后台运行 -p portId:portId:指定端口
查看所有镜像|docker ps -a|
启动已存在的镜像|docker start imageId/imageName|
停止容器|docker stop imageId|
重启容器|docker restart imageId|
进入容器|docker attach|exec imageId|attach类似于全局bash
删除容器|docker rm -f imageId|
删除不在使用的镜像|docker image prune|-a:全部

#### Dockerfile书写原则
> Dockerfile是构建docker镜像的一种方式，推荐作为生产环境构建镜像的方式。

1. 单一原则
容器的本质是进程，一个容器代表一个进程。因此不同功能的应用应该拆分为不同的容器，每个容器负责单一的业务进程。
2. 提供注释信息
3. 保持容器最小化
4. 合理选择基础镜像
容器的核心是应用，因此只要基础镜像能够满足应用的运行环境即可。
5. 使用.dockerignore文件
.dockerignore文件允许我们在构建时忽略一些不需要参与构建的文件，从而提高构建效率。
6. 尽量使用构建缓存
- 每一条Dockerfile指令都会提交一个镜像层，下一条指令都是基于上一条指令进行构建。如果构建时发现构建的镜像层的父镜像层已经存在，且下一条指令使用了相同的指令即命中缓存。
7. 正确设置时区
从Docker Hub拉取的官方操作系统镜像大多数都是UTC时间，如果在容器中需要使用东八区时间，需要进行修改

8. 使用国内软件源加快镜像构建速度
9. 最小化镜像层数

#### Dockerfile编写建议
1. RUN指令在构建时将会生成一个镜像层并且执行RUN指令后面的内容
2. CMD和ENTRYPOINT都是容器运行的命令入口；区别在于ENTRYPOINT指令启动容器时，需要用--entrypoint参数才能覆盖Dockerfile中的ENTRYPONT指令；CMD设置的命令则可以被docker run后面的参数直接覆盖
3.CMD/ENTRYPOINT的使用有两种格式；1.json数组格式的exec模式 2.command param格式的shell模式。这两种格式的区别在于启动容器后1号进程的区别。exec模式启动的进程即为容器的1号进程，而shell模式启动的进程不是容器的1号进程。

3. add和copy的区别。add是将外部资源加载到容器中这样会扩大容器体积，copy是将本地文件向容器中拷贝，可以更好的利用构建缓存，有效减小镜像体积。
- 反面例子
```bash
  ADD http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz /tmp/
  RUN tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp
  RUN make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install
```
- 替代例子

```bash
  RUN wget -O /tmp/memtester-4.3.0.tar.gz http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz \
  && tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp \
  && make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install
```



### 基础概念

#### Namespace
> Docker是使用Linux的Namespace技术实现各种资源隔离；Namespace是Linux内核的一项功能，该功能对内核资源进行分区，使一组进程只能看到一组资源，而另一组进程看到另外一组资源。

- Linux提供的Namespace类型

Namespace名称|作用|内核版本
--|--|--
Mount(mnt)|隔离挂载点|2.4.19
Process ID(pid)|隔离进程ID|2.6.24
Network(net)|隔离网络设备，端口号等|2.6.29
Interprocess Communication (ipc)|隔离 System V IPC 和 POSIX message queues|2.6.19
UTS Namespace(uts)|隔离主机名和域名|2.6.19
User Namespace (user)|隔离用户和用户组|3.8
Control group (cgroup) Namespace|隔离 Cgroups 根目录|4.6
Time Namespace|隔离系统时间|5.6

- Namespace功能

1. Mount Namespace
> Mount Namespace是Linux内核实现的第一个Namespace,可以用来隔离不同的进程或进程组看到的挂载点。通俗地说，就是可以实现在不同的进程中看到不同的挂载目录。

2. PID Namespace
> PID Namespace的作用是用来隔离进程。在不同的 PID Namespace 中，进程可以拥有相同的 PID 号，利用 PID Namespace 可以实现每个容器的主进程为 1 号进程，而容器内的进程在主机上却拥有不同的PID。

3. UTS Namespace
> UTS Namespace 主要是用来隔离主机名的，它允许每个 UTS Namespace 拥有一个独立的主机名。

4. IPC Namespace
> IPC Namespace 主要是用来隔离进程间通信的。例如 PID Namespace 和 IPC Namespace 一起使用可以实现同一 IPC Namespace 内的进程彼此可以通信，不同 IPC Namespace 的进程却不能通信。

5. User Namespace
> User Namespace 主要是用来隔离用户和用户组的。一个比较典型的应用场景就是在主机上以非 root 用户运行的进程可以在一个单独的 User Namespace 中映射成 root 用户。使用 User Namespace 可以实现进程在容器内拥有 root 权限，而在主机上却只是普通用户。

6. Net Namespace
> Net Namespace是用来隔离网络设备、IP地址和端口等信息的

- 小结
> 当Docker新建一个容器时,会创建这6种Namespace，来进行资源隔离。Namespace是Linux内核提供的特性，该特性可以实现在同一个主机上对<B>进程ID</B>、<B>主机名</B>、用户ID、<B>文件名</B>、<B>网络和进程间通信等资源的隔离</B>

#### cgrops
> cgrops是linux内核提供限制进程或进程组使用资源(如CPU、内存、磁盘IO)的功能

- cgrops功能
1. 资源限制
> 限制资源的使用量，例如我们可以通过限制某个业务的内存上限，从而保护主机其他业务的安全运行。
2. 优先级控制
> 不同的组可以有不同的资源（ CPU 、磁盘 IO 等）使用优先级
3. 审计
> 计算控制组的资源使用情况
4. 控制
> 控制进程的挂起或恢复

> Docker 创建容器时，Docker 会根据启动容器的参数，在对应的 cgroups 子系统下创建以<B>容器ID</B>为名称的目录, 然后根据容器启动时设置的资源限制参数, 修改对应的 cgroups 子系统资源限制文件, 从而达到资源限制的效果。


#### Docker组件的构成
Docker整体架构采用C/S架构，主要由客户端/服务端两大部分组成。客户端负责发送指令，服务端负责执行指令。

- Docker组件
1. docker
> docker是Docker客户端的一个完整实现，docker 组件向服务端发送请求后，服务端根据请求执行具体的动作并将结果返回给 docker，docker 解析服务端的返回结果，并将结果通过命令行标准输出展示给用户。

2. dockerd
> dockerd 是 Docker 服务端的后台常驻进程，用来接收客户端发送的请求，执行具体的处理任务，处理完成后将结果返回给客户端

3. docker-init
> docker-init是作为1号进程管理容器内的子进程

4. docker-proxy
> docker-proxy主要是用来做端口映射。当我们使用 docker run 命令启动容器时，如果使用了 -p 参数，docker-proxy 组件就会把容器内相应的端口映射到主机上来，底层是依赖于 iptables 实现的。

5. containerd
> containerd负责容器生命周期的管理、镜像的管理、dockerd的请求、管理存储、网络等相关资源。是容器标准化后的产物

6. containerd-shim
> 作为containerd和真正的容器进程的中间层

7. runc
> runc 是一个标准的 OCI 容器运行时的实现，它是一个命令行工具，可以直接用来创建和运行容器。




### 数据管理



### 网络管理

### 容器管理

### 镜像管理

