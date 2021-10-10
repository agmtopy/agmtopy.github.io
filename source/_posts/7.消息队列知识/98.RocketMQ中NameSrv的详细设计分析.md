---
title: RocketMQ中NameSrv的详细设计分析
date: 2021-06-11 22:57:56
categories: 消息队列
tags:
  - RocketMQ
---

# RocketMQ中NameSrv的详细设计分析

## 设计目标
NameSrv是RoctetMQ项目下的一个模块，作为RockerMQ中的轻型注册中心,只负责与Topic有关的功能。
使用NameSrv来替代ZK等注册中心主要是有两个好处:
1. 减少整体复杂性
  一个分布式系统强依赖另外一个分布式系统，增加了整个系统的复杂性(整体代码复杂性、运维的复杂性);
  使用内置的轻量级注册中心,就可以消除原来与ZK等第三方注册中心的各种协议适配;
  Namesrv中只需要开发与Topic有关的业务场景;
  系统维护时也不用在考虑第三方系统的处理机制;
2. 扩展能力
  RocketMQ从设计初就考虑过在嵌入式设备上进行部署的能力,因此采用Namesrv的设计能最大限度的掌握系统

目前Kafka在这方面做的更好，在V2.8之后不但去掉了ZK，并且采用broker集群中通过选举的方式选出leader节点来管理Topic、broker、consumer

## 基础功能

- 整体架构
![整体架构](https://github.com/apache/rocketmq/raw/master/docs/cn/image/rocketmq_architecture_3.png)
Namesrv集群是无状态的设计,每个组件(Broker、Producer、Consumer)都会向每一个Namesrv进行请求,因此Namesrv是选择了可用性

- 功能解析
1. Topic路由管理
2. Remoting远程服务
3. 定时任务
4. KV管理模块


## 源码分析

### Namesrv时序图
[![Namesrv执行流程](https://z3.ax1x.com/2021/06/16/2jkklQ.png)](https://imgtu.com/i/2jkklQ)


### NamesrvStartup

#### main()
```kotlin 
    /**
     * 主函数入口
     */
    fun main(args: Array<String>) {
        main0(args)
    }

    /**
     * 具体执行逻辑:避免在main函数中添加过多的逻辑
     */
    fun main0(args: Array<String>) {
        try {
            var controller = createNamesrvController(args)
            start(controller)
            val tip =
                "The Name Server boot success. serializeType=" + RemotingCommand.getSerializeTypeConfigInThisServer()
            log!!.info(tip);
            println(tip)
        } catch (e: Throwable) {

        }
    }

    /**
     * 初始化NamesrvController
     */
    fun createNamesrvController(args: Array<String>): NamesrvController {
      //去除TLS和apache.commons的版本
      return NamesrvController(NamesrvConfig(), NettyServerConfig())
    }
```

#### start()
```kotlin
    /**
      * 启动NamesrvController
      */
    fun start(controller: NamesrvController){
        if (controller == null) {
            throw IllegalArgumentException("NamesrvController is null")
        }

        //NamesrvController执行初始化,如果失败时退出进程
        if(controller.initialize()){
            controller.shutdown()
        }
        controller.start()
    }
```
NamesrvStartup启动时候主要做了以下几个步骤：
  1. 创建NamesrvController
  2. 处理TLS和加载Namesrv配置文件
  3. 启动NamesrvController
  4. 关闭NamesrvController

- NamesrvController
NamesrvController主要逻辑很为两个部分初始化和执行
1. 初始化
  - 初始化那些东西
  1. 配置
  2. 服务
  3. 线程池
  4. 自省

2. 加载配置文件

## 扩展思考

## 参考资料
[rocketmq](https://rocketmq.apache.org/)
[rocketmq-github](https://github.com/apache/rocketmq)
[Apache Kafka Made Simple: A First Glimpse of a Kafka Without ZooKeeper](https://www.confluent.io/blog/kafka-without-zookeeper-a-sneak-peek/)