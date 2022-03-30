---
layout: gitlab
title: GitLab CI/CD与GitHub Actions的介绍和使用
date: 2022-03-25 23:27:37
categories: 杂记
tags:
  - gitlab
---

# GitLab CI/CD与GitHub Actions的介绍和使用

本篇日志主要是用来记录<B>GitLab</B>的CI/CD与<B>GitHub Actions</B>的介绍和使用。先从操作GitLab的CI/CD开始


## GitLab CI/CD

GitLab支持多种CI方式,本身有Ai DevOps的支持,如下所示

![GitLab AI DevOps](https://github.com/agmtopy/noteBook/blob/master/png/gitlab_ai_devops.jpg?raw=true)



启动Ai DevOps后gitlab会根据项目的语言使用一套预设的模板,具体可以参考[GitLab Ai DevOps](https://docs.gitlab.com/ee/topics/autodevops/index.html)

在我们的演示项目中,我们可以选择开启Ai DevOps,但是实际并不会使用,而是使用我们项目中自定义的<B>.gitlab-ci.yml</B>文件

接下来介绍,让我们开始学习如何自定义GitLab的CI/CD与gradle结合使用.

- 前置条件:
    - 项目地址:https://github.com/agmtopy/testcontainers-simple
- 步骤:
    - clone 项目到本地
    - 修改.gitlab-ci.yml文件
    - 上传到GitLab
    - 执行构建

下面来详细的做一下步骤二和步骤四


### 修改.gitlab-ci.yml文件

```yml

services:
  - name: docker:dind         # Testcontainers需要docker in docker
    command: ["--tls=false"]  # 禁用tls以避免docker启动中断

variables:
  DOCKER_HOST: "tcp://docker:2375"
  DOCKER_TLS_CERTDIR: ""
  DOCKER_DRIVER: overlay2
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"

stages:          # 作业的阶段列表及其执行顺序
  - build
  - test
  - deploy

build-job:       # 此作业在构建阶段运行，构建阶段是最先执行的
  image: gradle:7.4.1-jdk11-alpine
  stage: build
  script:
    - echo "开始编译代码..."
    - gradle clean compileJava
    - echo "编译代码完成..."

test-job:        # 此作业在测试阶段运行，测试阶段在第二阶段执行的
  image: gradle:7.4.1-jdk11-alpine
  stage: test
  script:
    -  echo "开始测试代码..."
    - gradle test --info
    - echo "测试代码完成..."

deploy-job:      # 此作业在部署阶段运行
  stage: deploy  # 只有当之前两个阶段都成功完成时，才会执行部署的。
  script:
    - echo "开始部署代码..."
    - echo "部署代码完成.."

```


在上面这个.gitlab-ci.yml文件中,有一个稍微特殊一点的地方在于,由于我们的代码中使用到了testcontainers,所以我们需要在.gitlab-ci.yml文件中添加一个service,这个service是用来启动testcontainers相关容器的,具体可以参考[安装 Docker 镜像并启动容器](https://docs.gitlab.com/runner/install/docker.html#install-the-docker-image-and-start-the-container)

然后我们定义了三个阶段:build,test,deploy,这三个阶段的执行顺序是:build-job,test-job,deploy-job,这样我们就可以在GitLab中按照顺序执行这三个阶段的作业了,在流水线中我们也可以看到

![流水线](https://github.com/agmtopy/noteBook/blob/master/png/%E6%B5%81%E6%B0%B4%E7%BA%BF.jpg?raw=true)

由于我们没有资源,不能进行deploy,在这个阶段实际上只是打印一下,之前的build-job和test-job阶段分别是执行:
    - gradle clean compileJava
    - gradle test --info


![执行结果](https://github.com/agmtopy/noteBook/blob/master/png/%E6%89%A7%E8%A1%8C%E7%BB%93%E6%9E%9C.jpg?raw=true)

通过在流水线中点击相关的阶段,可以看到详细的日志

![执行日志](https://github.com/agmtopy/noteBook/blob/master/png/%E6%89%A7%E8%A1%8C%E6%97%A5%E5%BF%97.jpg?raw=true)


## GitHub Actions

GitHub Actions是github推出的一套发现、创建和共享操作以执行您喜欢的任何作业（包括 CI/CD），并将操作合并到完全自定义的工作流程。

### 创建Actions

在Actions中,选择对应的模板进行创建

![创建workflows](https://github.com/agmtopy/noteBook/blob/master/png/GitHub_Actions_1.jpg?raw=true)

![选择模板](https://github.com/agmtopy/noteBook/blob/master/png/GitHub_Actions_2.jpg?raw=true)

![模板](https://github.com/agmtopy/noteBook/blob/master/png/GitHub_Actions_3.jpg?raw=true)


### 执行Actions

创建完成Actions后,会自动运行.运行结果可以在workflow中查看

![执行Actions](https://github.com/agmtopy/noteBook/blob/master/png/GitHub_Actions_4.jpg?raw=true)

![执行结果](https://github.com/agmtopy/noteBook/blob/master/png/GitHub_Actions_5.jpg?raw=true)

![执行日志](https://github.com/agmtopy/noteBook/blob/master/png/GitHub_Actions_5.jpg?raw=true)


注意:

  - 编译gradle项目出现'Error: Gradle script '/home/runner/work/testcontainers-simple/testcontainers-simple/gradlew' is not executable.'的问题是由于我们项目中的gradlew文件不是可执行的,所以需要将其设置为可执行的,在项目中执行<B>git update-index --chmod=+x gradlew</B>即可
  

  


### Actions配置解释

- main.yml

```yml文件
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.
# This workflow will build a Java project with Gradle and cache/restore any dependencies to improve the workflow execution time
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-gradle

name: Java CI with Gradle

on: push        # 触发事件

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@0d13054264b0bb894ded474f08ebb30921341cee
      with:
        arguments: build
```

github的CI/CD配置就比gitlab的配置看起来要清爽很多,主要是分为三个部分:

- name
  name主要是用来标识这个workflows的名称的

- on
  on主要是用来标识触发Actions的动作的


- jobs
  jobs又很为两个部分:
    - runs-on
      标识运行的基础环境
    - steps
      标识运行的步骤,其中with.arguments是用来指定gradle的参数的

## 总结

GitLab和GitHub的CI/CD在使用上都是比较简单的,只需要使用到专用的配置文件就可以触发,需要主要一点的是GitLab的配置文件中需要指定Docker in Docker配置,
而GitHub Actions使用上要比GitLab的.gitlab-ci.yml要方便,可以直接从marketplace下载已经预设好的模板,直接进行使用.配置文件中也不需要指定运行环境

## 参考文档

[gitlab install](https://about.gitlab.com/install/)
[安装和使用GitLab](https://help.aliyun.com/document_detail/52857.html)
[GitLab CI/CD 介绍和使用](http://blinkfox.com/2018/11/22/ruan-jian-gong-ju/devops/gitlab-ci-jie-shao-he-shi-yong/#toc-heading-20)
[CI/CD](https://zq99299.github.io/note-combat/gitlab/cicd/)
[GitHub Actions 快速开始](https://docs.github.com/cn/actions/quickstart)