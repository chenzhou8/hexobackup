---
title: 网页启动本地Activity
date: 2018-02-13 11:02:28
toc: true
categories: 移动开发
---

# 前言
Intent这个类在开发中是很常用的类，代表了着一个意图（获取理解为目标、目的），首先我们需要明确一点的就是：任何一个浏览器链接都是一个隐式意图，打开一个浏览器的方式无非就是显式意图和隐式意图，所以我们配置过滤器即可！


# 示例
首先，工程目录如图：

![](https://s2.ax1x.com/2019/04/30/EGmf81.png)

MainActivity和布局文件都不用改，关键是manifests文件中LocalAppAty的配置:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.xpu.launchlocalapp">
 
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
 
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".LocalAppAty" android:label="LocalAppAty">
            <intent-filter>
                <!--可以被浏览器启动的Activity-->
                <category android:name="android.intent.category.BROWSABLE"></category>
                <category android:name="android.intent.category.DEFAULT"></category>
 
                <action android:name="android.intent.action.VIEW"></action>
                <data android:scheme="app" ></data>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

JavaWeb工程如下，一个很简单的标签：

![](https://s2.ax1x.com/2019/04/30/EGmbUH.png)

一般安卓模拟器要访问本机的ip地址，使用10.0.0.2,端口号还是与你的服务器一致，我的是8080：

![](https://s2.ax1x.com/2019/04/30/EGmzKf.png)

成功开启：

![](https://s2.ax1x.com/2019/04/30/EGnCVg.png)

同时获取到了启动该Activity的信息来源：

![](https://s2.ax1x.com/2019/04/30/EGnPaQ.png)



