---
title: Netty入门基础一
date: 2020-07-29 22:41:24
categories: Netty
tags:
  - Netty
---

# Netty入门基础

## Netty是什么？
> Netty 是一个异步事件驱动的网络应用框架，用于快速开发可维护的高性能服务器和客户端。

Netty针对java nio做了封装和改进

## Netty 版本的Hello world！

- NettyServer.java
```java
public class NettyServer {
    public static void main(String[] args) {
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        NioEventLoopGroup boosGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        serverBootstrap.group(boosGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new StringDecoder());
                        ch.pipeline().addLast(new SimpleChannelInboundHandler<String>() {
                            @Override
                            protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
                                System.out.println(msg);
                            }
                        });
                    }
                })
                .bind(8000);
    }
}
```

- NettyClient.java

```java
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        Bootstrap bootstrap = new Bootstrap();
        NioEventLoopGroup group = new NioEventLoopGroup();

        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>(){
                    @Override
                    protected void initChannel(Channel ch) throws Exception {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                });
        Channel channel = bootstrap.connect("127.0.0.1",8000).channel();

        while (true){
            channel.writeAndFlush(LocalDateTime.now() + ": hello world!");
            TimeUnit.SECONDS.sleep(2L);
        }
    }
}
```



