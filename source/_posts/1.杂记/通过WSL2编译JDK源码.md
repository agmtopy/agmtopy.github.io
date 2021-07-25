---
title: 通过WSL2编译JDK源码
date: 2021-06-19 19:39:54
categories: 杂记
tags:
  - WSL
---

# 通过WSL2编译JDK源码

WSL的全称是'Windows Subsystem for Linux',通过在系统层面对Linux内核进行支持,WSL1只是部分支持Linux内核而WSL2支持完整的Linux内核。不但可以通过WSL运行Linux内核，甚至可以将Windows Docker指定通过WSL2来进行远行

## WSL1 -> WSL2

- 安装Linux发行版
WSL1只需要开启Linux子系统并下载linux发行版即可,linux发行版默认是只能通过Windows Store进行下载，也可以通过下载安装包(msi)进行安装,手动选择安装包地址如下(https://docs.microsoft.com/zh-cn/windows/wsl/install-manual)

- 切换WSL2
```bash
wsl --set-default-version 2
```

- 启动WSL
1. 应用程序直接启动安装Linux发行版
2. 'wsl -l -v' 检查WSL版本,VERSION -> 2即可
[![WSL2](https://z3.ax1x.com/2021/06/19/RPsbr9.png)](https://imgtu.com/i/RPsbr9)


## 编译JDK
严谨的说法是编译OpenJDK

### 下载JDK源代码
推荐直接下载,下面是JDK11的源代码地址,直接选择zip file进行下载,也可以选择不同的版本
https://jdk.java.net/java-se-ri/11

### 准备JDK编译环境

#### 安装必要程序
- 推荐先先切换软件源

```bash
    sudo sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
    sudo apt update -y
```
- 安装必要软件

```bash
sudo apt-get install -y curl ssh zip unzip ant git build-essential ccache cpio g++ gcc gdb libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev libcups2-dev libfreetype6-dev libasound2-dev libelf-dev ccache libfontconfig1-dev autoconf
```

#### 安装BootStrap JDK
BootStrap JDK的作用是编译JDK源代码中的java代码,默认要比编译的JDK小一个版本

```bash
sudo apt install openjdk-10-jdk
java -version
```
#### 解压JDK源代码
由于我比较喜欢使用VS code,VS code中已经集成了连接WSL的插件，因此只需要将JDK源代码直接拖拽到Linux的目录下即可。也可以通过WSL进行复制
得到JDK源代码后就可以进行解压

#### 编译JDK源代码
```bash 
# 准备配置(警告不作为异常抛出)
bash ./configure --disable-warnings-as-errors

make all

# 编译失败时推荐先进行clean,在重新进行编译
make clean
```

#### 异常处理
1. 在构建低版本的JDK时可能出现OS版本不支持
修改./hotspot/make/linux/Makefile
```bash 
    # 5% 内核版本
    SUPPORTED_OS_VERSION = 2.4% 2.5% 2.6% 2.7% 3% 4%
```







