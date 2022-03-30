---
title: Docker基础概念入门(二)
date: 2020-11-29 21:32:37
categories: 杂记
tags:
  - docker
---

## Dockerfile文件

> Dockerfile文件是用来构建镜像的文本文件，文本文件中包含了一系列构建镜像所需的指令和说明

## java基础环境

- Dockerfile

```bash
# 基础镜像版本
FROM java:latest

# 设置工作目录
WORKDIR /app

# 复制初始文件到工作目录中
COPY . /app

# 设置Java环境变量
ENV PATH=$PATH:$JAVA_HOME/bin
ENV JRE_HOME=${JAVA_HOME}/jre
ENV CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib

# 编译
RUN ["/usr/lib/jvm/java-8-openjdk-amd64/bin/javac","Docker_java.java"]

# 运行
ENTRYPOINT ["/usr/lib/jvm/java-8-openjdk-amd64/bin/java", "Docker_java"]
```

Dockerfile文件需要在当前工作目录下编写，Docker_java.java文件也需要在当前工作目录下

- Docker_java.java
```java
import java.util.*;

public class Docker_java{
    public static void main(String[] args){
        System.out.println("Hello Docker!");
        System.out.println(new Date());
    }
}
```

- build命令和启动命令

```bash
# -rm 表示构建完成后删除中间镜像
# -f  表示指定Dockerfile文件,默认为当前目录下的Dockerfile文件 
# -t  表示对镜像打tag
docker build --rm -f "Dockerfile" -t docker_java:0.0.1

#运行命令
```
    docker run docker_java:0.0.1
```

## mysql环境
> 构建一个账号名称为test,密码为test并且自定义初始化sqlmysql容器

- Dockerfile

```bash
#基础版本
FROM mysql:5.7.22

#环境变量
ENV TZ=Asia/Shanghai \
    MYSQL_DATABASE=test \
    MYSQL_USER=test \
    MYSQL_PASSWORD=test \
    MYSQL_ROOT_PASSWORD=test

#将sql目录下的文件全部拷贝到目标文件夹下
COPY sql/. /docker-entrypoint-initdb.d
```

- init.sql

```sql
    CREATE DATABASE IF NOT EXISTS `test`;
    USE `test`;
    DROP TABLE IF EXISTS `table_a`;
    CREATE TABLE `table_a` (
    `sync_pk` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL ,
    `hos_code` varchar(64) COLLATE utf8mb4_unicode_ci DEFAULT NULL ,
    `create_time` varchar(19) COLLATE utf8mb4_unicode_ci DEFAULT NULL ,
    `study_iuid` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL ,
    `study_content` mediumtext COLLATE utf8mb4_unicode_ci ,
    `study_state` int(255) DEFAULT NULL,
    `remark` varchar(1024) COLLATE utf8mb4_unicode_ci DEFAULT NULL ,
    PRIMARY KEY (`sync_pk`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ;
```

- 构建命令

```bash
    docker build --rm -f "Dockerfile" -t mysqlc:0.0.1 .
```

- 运行命令
```bash
    #与mysql启动一致
    docker run -itd --name mysql_local mysqlc:0.0.1 
```