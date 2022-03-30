---
title: 《java8实战》读书笔记
date: 2020-06-20 13:53:14
categories: 杂记
tags:
  - java
---

# 《java8实战》读书笔记

主要是关于java8的一些笔记。

## 基础
### java8中设计的三个编程概念
1. **流处理**
2. **函数传递**
3. 并行与共享的可变数据


### 函数传递
> 行为参数化(函数传递)就是可以帮助你处理频繁变更的需求的一种软件开发模式

- 例1:用Comparator来排序

```java
    List<Integer> ls = Lists.newArrayList(12,13,14,15,16);
    //Comparator进行排序
    ls.sort((Integer i1,Integer i2) -> -i1.compareTo(i2));
    System.out.println(ls);
```

- 例2:用Runable执行代码块

```java
    List<Integer> ls = Lists.newArrayList(12,13,14,15,16);
    //Comparator进行排序
    ls.sort((Integer i1,Integer i2) -> -i1.compareTo(i2));
    System.out.println(ls);
```

### lambda表达式

主要记录一下**anyMatch**、**allMatch**、**noneMatch**、**findAny**

- **anyMatch**
```java
    List<Integer> intList = Lists.newArrayList(1,2,3,4,5);

    //全部大于
    if(intList.stream().allMatch((item) -> item > 4)){
        System.out.println("ls集合存在大于4的数字!");
    }
    //存在大于
    if(intList.stream().anyMatch((item) -> item > 2)){
        System.out.println("ls集合存在大于5的数字!");
    }
```

- **findAny**
```java
    
    Optional<Integer> optional = intList.stream().findAny();
```
**findAny**这里有一个需要注意的是返回的是steam流中的某一个元素，并不一定是第一个元素。在串行的情况下可能返回的是第一个元素，但是不保证并行的情况下仍然是第一个元素。


### Optional

> Optional<T>类（java.util.Optional）是一个容器类，代表一个值存在或不存在。


- isPresent
表示如果元素存在返回true，反之返回false

- ifPresent
表示如果元素存在执行方法,不存在不执行

- get
特别要注意一下，在不存在值的情况下会抛出异常

- orElse()
当值不存在时，返回默认值


### reduce操作
> 合并操作

```java
    List<Integer> reduceSumls = Arrays.asList(1,2,3,4);
    //第一种写法
    Optional<Integer> sum = reduceSumls.stream().reduce((Integer a, Integer b) -> a +b);
    //第二种写法
    Optional<Integer> sum = reduceSumls.stream().reduce(0,Integer::sum);
    System.out.println("reduceSum = " + sum.get());
```
