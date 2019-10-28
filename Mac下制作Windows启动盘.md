---
title: Mac下制作Windows启动盘
date: 2017-06-17 13:16:27
tags:
---

## 下载Windows镜像

https://msdn.itellyou.cn/ ,在这个网站可以找到你要下载的镜像文件，当然还有其他的下载链接

## 双击ISO文件

双击你下载的ISO文件，这时你相当于将镜像挂载到了自己的电脑上，应该出现如下所示的情况。

![](https://s2.ax1x.com/2019/06/17/VH1MPH.png)

![](https://s2.ax1x.com/2019/06/17/VH1tZ8.png)

注意挂载后的文件夹的名字，如我的Win10镜像挂载之后的名字就是J_CCSA_X64FRE_ZH-CN_DV5。

## 将挂载后的文件拷贝入U盘

```
diskutil list
```

会返回当前所有Volume的列表，找到你的U盘，这里为disk2

![](https://s2.ax1x.com/2019/06/17/VH1yLV.png)

将U盘格式化为MS-DOS格式输入下列命令并将disk#更改为你U盘的序号，比如上述列子就改为disk2：

```
diskutil eraseDisk MS-DOS "WINDOWS10" MBR disk2
```

将镜像文件拷贝进U盘，输入下列命令并将VolumeName换成你的镜像的名字，比如我就换成J_CCSA_X64FRE_ZH-CN_DV5

```
cp -rp /Volumes/VolumeName/* /Volumes/WINDOWS10/ 
```

当Terminal里出现新的一行带“~”的内容时，启动盘就制作成功了！



转载自：[http://www.xanderxu.com/post/mac_windows.html](http://www.xanderxu.com/post/mac_windows.html)