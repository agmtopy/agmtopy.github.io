---
title: git多环境配置
date: 2020-02-21 22:00:49
categories: 杂记
tags:
  - 环境配置
---
疫情期间，需要在家远程办公，因此需要在电脑上配置两套git环境分别是github、gitlab的。在配置过程中遇到了一些阻碍，特此记录下来。

### 步骤一：生成SSH-Key

```bash
    ssh-keygen -t rsa -C "email@xx.com" -f ~/.ssh/id_rsa #(或gitlab_rsa)
```
其中email@xx.com为github或gitlab注册的邮箱
id_rsa或gitlab_rsa为生成key文件的名称，默认为id_rsa，使用别名时需要进行步骤三

- 分别生成github和gitla的SSH-key

### 步骤二：设置Server端公钥

```bash
    cat ~/.ssh/id_rsa.pub #(或gitlab_rsa)
```
将获取到的公钥设置到git服务端，也可以用编辑器打开~/.ssh/id_rsa.pub(或gitlab_rsa)将值复制设置到git服务端

### 步骤三：添加私钥

```bash
    ssh-add ~/.ssh/id_rsa #(或gitlab_rsa)
```
分别添加不同环境的私钥

### 步骤三：配置多环境的config

在.ssh/下创建config文件

```bash
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
# gitlab
Host gitlab.com
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab_rsa
```

### 步骤四：测试

```bash
    ssh -T git@github.com
```
使用命令进行测试

```bash
Hi userName! You've successfully authenticated, but GitHub does not provide shell access.
```
这样的提示表示成功


### 异常处理

- git@github.com: Permission denied (publickey). 
1.是否修改过github.com的host
2.是否添加私钥

### 参考
[git 配置多个SSH-Key](https://my.oschina.net/stefanzhlg/blog/529403)
[解决Permission denied (publickey).](https://www.cnblogs.com/lxwphp/p/7884700.html)