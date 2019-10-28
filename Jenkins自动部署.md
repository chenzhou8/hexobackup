---
title: Jenkins自动部署
date: 2019-06-03 19:49:20
toc: true
categories: 工程自动化
---

事先确保服务器上有Java（大于1.6）环境与Maven环境

## 安装Jenkins

这里是Jenkins的文档  https://jenkins.io/zh/doc/ 

这里是Jenkins的下载地址https://jenkins.io/zh/download/ 

**最好的方式应该是自己去下载Jenkins的Jar包，直接像运行jar包即可**

```
nohup java -jar jenkins.war --httpPort=80
```

这样直接访问`172.16.45.112:8080` 即可！

<!-- more -->

## 配置Jenkins

成功开启服务后应该是下面这个样子：

![](https://s2.ax1x.com/2019/06/03/VYCP5F.png)

登录密码：

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

我直接选择推荐插件了：
![](VYCKUO.png)

## Jenkins自动构建

原文参考  [《Jenkins自动构建》](https://blog.csdn.net/boling_cavalry/article/details/78943061)

当我们提交代码到GitHub后，可以在Jenkins上执行构建，但是每次都要动手去执行略显麻烦，今天我们就来实战Jenkins的自动构建功能，每次提交代码到GitHub后，Jenkins会进行自动构建！

### 注意点

GitHub收到提交的代码后要主动通知Jenkins，所以Jenkins所在服务器一定要有外网IP，否则GitHub无法访问，我的Jenkins服务器是部署在腾讯云的云主机上，带有外网IP；



### 整个流程

* GitHub上准备一个spring boot的web工程；
* GitHub上配置Jenkins的webhook地址；
* 在GitHub上创建一个access token，Jenkins做一些需要权限的操作的时候就用这个access token去鉴权；
* Jenkins安装GitHub Plugin插件；
* Jenkins配置GitHub访问权限；
* Jenkins上创建一个构建项目，对应的源码是步骤1中的web工程；
* 修改web工程的源码，并提交到GitHub上；
* 检查Jenkins的构建项目是否被触发自动构建，构建成功后，下载工程运行，看是不是基于最新的代码构建的