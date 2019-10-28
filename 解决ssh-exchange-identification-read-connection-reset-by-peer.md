---
title: '解决ssh_exchange_identification:read connection reset by peer'
date: 2019-05-19 22:03:53
toc: true
categories: Linux应用
---

### 问题
最近好久没连接过自己的云服务器了，但是的时候使用ssh访问服务器时出现：`ssh_exchange_identification: read: Connection reset by peer`这样的连接错误

### 原因
* 服务器防火墙限定
* 是否达到ssh的最大连接数，超过之后会服务器端会拒绝新的连接，直到有新的连接释放出来
* /etc/hosts.allow和/etc/hosts.deny配置文件限定ip登录

### 解决
先关闭防火墙，尤其是云服务厂商为你设定的防火墙，具体解决方式要参考云服务器厂商。看看网络状态，看看Linux是否运行着shhd服务，如果没有那么有可能是连ssh服务程序都没有安装，应该先安装ssh服务器才可以。接着如果还是不行的话：


```
vim /etc/hosts.allow
```
追加上：

```
sshd: ALL
```
接着重启shh服务

```
service sshd restart
```
如果这还不行留言讨论，OK反正我的是可以正常连接了!

<!-- more -->