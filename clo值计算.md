---
title: clo值计算
date: 2019-06-13 21:01:26
categories: 移动开发
toc: true
---

## 一、clo是什么

服装热阻是指反映服装保温性能的参数。其值与服装导热系数成反比。单位为Clo。1Clo=0.155m·K/W。各种服装的热阻值有实测数据可查用。它与周围环境温度、风速和人体散热量有密切关系，下面是研究简化后的计算式：

In(col) = 0.360 − 0.033ta + effectofseason + effectofbuildingtype

上面就是col值与温度、季节、建筑类型之间的关系。

季节和建筑类型影响的混合模型估计：

![](https://s2.ax1x.com/2019/06/15/VoILSH.png)

## 二、优化的计算col

于是，一个App诞生了：

![](https://s2.ax1x.com/2019/06/15/VoIOld.gif)

看起来还觉得莫名的好看（其实是因为我用了一个Android的UI库：XUI，地址是：），好了，接下来说说XUI的引入方式：https://github.com/xuexiangjys/XUI

## 三、XUI库的使用

build.gradle(Module.app)

```json
compileOptions {
  	sourceCompatibility JavaVersion.VERSION_1_8
  	targetCompatibility JavaVersion.VERSION_1_8
}

lintOptions {//程序在buid的时候，会执行lint检查，有任何的错误或者警告提示，都会终止构建
  	abortOnError false
}

dependencies {
		...
		implementation 'com.github.xuexiangjys:XUI:1.0.3'
 		...
}
```

Build.gradle(Project: BuildEnvironment)

```json
allprojects {
    repositories {
        maven { url "https://jitpack.io" }
        google()
        jcenter()
    }
}
```

下载官方示例程序，解压后就得到了使用示例代码：

![](https://s2.ax1x.com/2019/06/15/VoIbfe.png)

新建一个自己的工程，引入XUI库即可：

修改styles.xml (parent="XUITheme.Phone"):

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="XUITheme.Phone">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>
```

新建一个MyApp.java 用来初始化自己的App以及初始化XUI的框架：

```java
package edu.xpu.buildenvironment;

import android.app.Application;
import android.content.Context;

import com.xuexiang.xui.XUI;

public class MyApp extends Application {
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
    }

    @Override
    public void onCreate() {
        super.onCreate();
        initUI();
    }

    private void initUI() {
        XUI.init(this);
        XUI.debug(BuildConfig.DEBUG);
    }
}
```

接下来，需要配置AndroidManifest.xml

```xml
<application
    android:name=".MyApp"
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
</application>
```

好了，接下来对照着XUI给的Demo工程就可以大致知道控件的使用方式了,比如我用到的是一个标尺(就是那个温度选择器)，就可以得知大致API的使用了：

![](https://s2.ax1x.com/2019/06/15/VoIX6A.png)

至于最终结果的实时计算呢，恐怕才是最近最大的收获，之前一直不太明白为啥不用点击某个计算按钮之后才执行某个运算过程，原来以前自己的理解是出现了大偏差，所谓事件其实不只是点击事件，想起之前学习JS的时候学习了好多事件，鼠标的点击事件，鼠标按下的事件，鼠标移动的事件，鼠标松开的事件，键盘按下的事件，这些东西统统都是事件，也就是说我们只要监听相应的事件，设置响应的触发器(或者说叫做事件回调)就可以了，我之前居然一直以为是拿多线程一直不停的算，今天算是想明白了，原来是把事件臆想成点击事件了，哈哈，好吧，顺便把这个 "尺子" 控件的事件回调写写：

```java
rulerView.setOnChooseResultListener(new RulerView.OnChooseResultListener() {
  @Override
  public void onEndResult(String result) {
    Log.i("MainActivity", "onEndResult:Result = " + result);
  }

  @Override
  public void onScrollResult(String result) {
    Log.i("MainActivity", "onScrollResult:Result = " + result);
    //执行计算的函数
    .....
  }
});
```

Github开源项目地址：

https://github.com/xuexiangjys/XUI