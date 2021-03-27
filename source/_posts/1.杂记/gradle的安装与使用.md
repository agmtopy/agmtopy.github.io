---
title: gradle的安装与使用
date: 2020-06-16 22:49:05
categories: 杂记
tags:
  - gradle
  - 编译工具
---

# gradle的安装与使用

> 由于新的项目大多是使用gradle进行依赖管理，因此在这里将其用法做一下总结

## 安装

主要步骤是：
1. 下载安装包
2. 配置环境变量
3. 查看版本号
> gradle -v
- 显示对应版本号表示安装成功
![NkXXFA.png](https://s1.ax1x.com/2020/06/16/NkXXFA.png)

## 基础知识

gradle是一个基于Ant和Maven的理念的项目自动化构建工具。基于Groovy的特定领域语言（DSL）来声明项目设置。



### gradle与maven的对比
1. 灵活性更高
2. 性能更好
3. 依赖管理

### gradle构建生命周期
gradle构建系统的生命周期可以分为：**初始化**、**配置**、**运行**三个阶段

![NQ6IOg.png](https://s1.ax1x.com/2020/06/20/NQ6IOg.png)

1. 初始化阶段
此阶段读取根目录下的**setting.gradle**中的include信息，来决定构建哪些工程。以spring为例

```Groovy

## 省略若干项目
include "spring-websocket"
include "spring-framework-bom"

// Exposes gradle buildSrc for IDE support
include "buildSrc"
rootProject.children.find{ it.name == "buildSrc" }.name = "spring-build-src"

rootProject.name = "spring"
rootProject.children.each {project ->
	project.buildFileName = "${project.name}.gradle"
}
```


2. 配置阶段
此阶段主要解析每个项目中的**build.gradle**配置文件、**处理依赖关系**、**执行顺序**
以**spring beans**和**spring-core**两个项目为例

- **spring-beans**的gradle文件

```Groovy
//本项目的描述
description = "Spring Beans"
//插件
apply plugin: "groovy"
//依赖关系
dependencies {
    //依赖同一个项目下的spring-core模块
	compile(project(":spring-core"))
    //"javax.inject:javax.inject:1"等同与pom文件
    //<dependency>
    // <groupId>javax.inject</groupId>
    // <artifactId>javax.inject</artifactId>
    // <version>1</version>
    // <scope>compile</scope>
    //</dependency>
	optional("javax.inject:javax.inject:1")
	optional("org.yaml:snakeyaml:1.20")
    optional("org.codehaus.groovy:groovy-all:${groovyVersion}")
	optional("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")
	optional("org.jetbrains.kotlin:kotlin-stdlib:${kotlinVersion}")
    //只会在测试期间存在
	testCompile("org.apache.tomcat.embed:tomcat-embed-core:${tomcatVersion}")
}

```

- **spring-core**的gradle文件
```Groovy
dependencies {
    //省略...
	cglib("cglib:cglib:${cglibVersion}@jar")
	objenesis("org.objenesis:objenesis:${objenesisVersion}@jar")
	jarjar("com.googlecode.jarjar:jarjar:1.3")
    //省略...
	compile(files(cglibRepackJar))
	compile(project(":spring-jcl"))
	optional("io.netty:netty-buffer")
	testCompile("com.fasterxml.woodstox:woodstox-core:5.0.3") {
		exclude group: "stax", module: "stax-api"
	}
}
```

3. 运行阶段
> 根据Gradle命令传递过来的任务名称，执行相关依赖任务。此阶段真正构建项目并执行项目下的各个任务。Gradle就会将这个任务链上的所有任务全部按依赖顺序执行一遍。


### gradle插件








### task的概念

**gradle**是通过task任务的模型进行驱动开发的，在一个gradle工程中将不同的构建工作打散成不同的task。task脚本采用Groovy语言进行编写。

- task定义示例

```gradle
  task helloword {
      println 'hello word!'
  }
```

- task依赖示例

```gradle

 //task显式调用
  task task1 {
        println 'hello word!'
  }

  task task2 {
      println 'hello task2!'
  }

  task1.dependsOn task2

  //task内依赖
  task task3(dependsOn: task2) {
      println 'hello task3!'
  }

```


- 跳过依赖

```Gradle

  compile.doFirst {
      // Here you would put arbitrary conditions in real life.
      // But this is used in an integration test so we want defined behavior.
      if (true) { throw new StopExecutionException() }
  }

```

### Gradle申明依赖

### 配置远程仓库

```gradle
repositories {
   maven {
      url "http://repo.mycompany.com/maven2"
   }
}
```





## 基础命令

- build 编译命令
> ./gradlew build --scan

表示编译项目并上传相应的编译数据到云端进行分析，本地分析的命令是

>  ./gradlew clean app:assembleDebug --profile
该命令会在**build/report**中生成对应的报告

- gradle  build
> gradle  build<br> 编译（编译过程中会进行单元测试）

- gradle test
> gradle test <br> 单元测试

- gradle test
> gradle build -x test <br> 编译时跳过单元测试

- gradle run
> gradle run <br> 直接运行项目

- gradle clean
> gradle clean 清空所有编译、打包生成的文件(即：清空build目录)



## build.gradle模板

```gradle
// 创建Java项目插件，提供了所有构建和测试Java程序所需项目结构
apply plugin: 'java'
// 引入Maven插件，以便Gradle可以引用Maven仓库的Jar包
apply plugin: 'maven'
// 进行打包插件
apply plugin: 'war'
// Web项目服务器插件,此处配置后，不需再单独配置Web服务器
apply plugin: 'jetty'
// Web项目开发插件
apply plugin: 'eclipse'

group = "com.suneee"
// 指定编译*.java文件的JDK版本
sourceCompatibility = 1.8
version = '1.0'
// Web项目示例工程
jar {
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
        // 题外话：Java项目，此处可以用来配置程序入口点
        // attributes 'Main-Class':'com.suneee.**'
    }
}
// 默认情况，Gradle没有定义任何资源库
repositories {
    // 使用Maven的中央存储库
    mavenCentral()
    // 使用Maven的本地存储库
    mavenLocal()
}
// 增加自定义的库
repositories {
    flatDir{
            // 项目目录下的lib文件夹
            dirs 'lib' 
        }
}
// 解决Gradle编译时出现： 编码GBK的不可映射字符
[compileJava, compileTestJava]*.options*.encoding = 'UTF-8'

// 解决项目Jar包依赖问题
dependencies {
    // 编译时，加入项目目录下lib文件
    compile fileTree(dir:'lib',include:'*.jar')
    compile 'c3p0:c3p0:0.9.1.2'
    compile 'mysql:mysql-connector-java:5.1.31' 
    compile 'org.springframework.data:spring-data-redis:1.3.4.RELEASE'
    compile group: 'com.google.code.gson', name: 'gson', version: '2.7'
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile "org.springframework:spring-test:4.1.0.RELEASE"
    testCompile group: 'junit', name: 'junit', version: '4.+'
    compile 'com.alibaba:dubbo:2.8.4a'
    compile 'org.apache.zookeeper:zookeeper:3.4.8'
    compile 'com.101tec:zkclient:0.8'
    compile 'log4j:log4j:1.2.17'
    compile group: 'javax.activation', name: 'activation', version: '1.1.1'
    compile group: 'javax.mail', name: 'mail', version: '1.4.7'
    compile group: 'org.quartz-scheduler', name: 'quartz', version: '1.7.2'
    compile group: 'aspectj', name: 'aspectjweaver', version: '1.5.4'
    compile group: 'org.hibernate', name: 'hibernate-core', version: '5.1.0.Final'
    compile group: 'org.hibernate', name: 'hibernate-entitymanager', version: '5.1.0.Final'
    compile group: 'org.springframework', name: 'spring-orm', version: '4.2.4.RELEASE'
    
}

test {
    systemProperties 'property': 'value'
}

// 发布项目导出的Jar包到指定仓库
uploadArchives {
    repositories {
       flatDir {
           dirs 'repos'
       }
    }
}

```

## 总结

- 查看task
> gradle task

- 编译-编译过程中会进行单元测试
> gradle build

- 单元测试
> gradle test

- 编译时跳过单元测试
> gradle build -x test

- 直接远行项目
> gradle run

- 清空build文件
> gradle clean

- 安装jar到maven仓库
> gradle install



## 参考资料
[官方下载地址](https://gradle.org/release-candidate/)
[Gradle学习笔记](https://www.cnblogs.com/shaloo/p/6096277.html)
[Gradle的安装与配置](https://www.cnblogs.com/NyanKoSenSei/p/11458953.html)
[【入门】Gradle的基本使用、在IDEA中的配置、常用命令](https://www.cnblogs.com/lucas1024/p/9533566.html)

