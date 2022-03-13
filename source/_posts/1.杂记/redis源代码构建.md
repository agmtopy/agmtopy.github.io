---
title: redis源代码构建
date: 2022-01-19 21:52:20
categories: 杂记
tags:
  - redis
---

# redis源代码构建

主要用来记录以下在WSL下编译和部署redis源码的过程

## 环境准备

- WSL
  wsl2搭配Ubuntu 18.04使用,这一项不是必须的
![wsl](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20220306.jpg)

- Clion
 JetBrains的C/C++ IDE,这一项也不是必须的

- 源代码
    https://github.com/redis/redis

## 构建过程

-  按照Linux编译工具

```bash
    apt update
    apt install git  cmake make gcc g++ gdb
```

- 构建release.h

```bash
    #授权
    chmod +x mkreleasehdr.sh
    #执行make
    make install
```

以上两个步骤就完成redis源代码的编译,接下来让我们把源代码导入CLion中进行debug


## 调试过程

- 导入项目

![导入项目](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20231623.jpg)


- 设置远程部署

1. 设置远程部署工具链

![工具链](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20231847.jpg)

2. 设置CMake 

![CMake](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20231912.jpg)

3. 设置MakeFile 

![MakeFile](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20232004.jpg)


- 设置调试器

![设置调试器](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20232030.jpg)

## debug

- 启动redis-server

![启动redis-server](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20232618.jpg)

- 设置断点

![设置断点](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-19%20235133.jpg)

以set为例,基本调用过程为
<B>server.c</B> -> <B>connection.c</B> -> <B>t_set.c</B>

## 总结
以上就是本地编译redis的过程,后续官网上还有编译挂载redis扩展库的过程和dockerfile的相关操作(未完待续!)