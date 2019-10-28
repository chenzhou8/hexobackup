---
title: 软硬链接与Linux文件系统
date: 2018-10-26 13:06:00
toc: true
categories: Linux应用
---

想要理解硬链接和软链接必须要了解一下什么是Linux的文件系统
## 文件分类
![](https://s2.ax1x.com/2019/05/07/EsQSJJ.png)
### 普通文件 (-)
这个不用说，常见的音频、视频、文本、可执行程序都是普通文件
### 目录文件 (d)
**如果是要查看目录，需要读权限；如果要进入目录，需要该目录具有可执行权限；如果要在目录里修改或者增加文件，那么需要写权限**
### 字符设备文件 (c\)
提供连续的数据流，应用程序可以顺序读取，不支持随机存取。
键盘、调制解调器等等都是字符设备文件，在你按键的时候系统只能一个一个从键盘上读取字符，这样的设备就是字符设备
### 块设备文件 (b)
应用程序可以随机访问设备数据，程序可自行确定读取数据的位置。硬盘、软盘、CD-ROM驱动器和闪存都是典型的块设备，应用程序可以寻址磁盘上的任何位置，并由此读取数据。
### 套接字文件 (s)
这类文件通常用在网络数据连接。可以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。第一个属性为 [s]，最常在 /var/run目录中看到这种文件类型 
### 链接文件 (l)
这个在本文后面将说到！
### 管道文件 (p\)
管道文件可以用于进程间通信，至于什么是管道我会在Linux进程间通信的博客中说到！

## 文件应该是两部分构成
文件信息+文件内容 ,**stat命令可以查看文件的具体信息**
```bash
[zcl@localhost ~]$ ll
total 12
drwxrwxr-x. 28 zcl zcl 4096 Oct 24 21:56 Code
drwxr-xr-x.  3 zcl zcl   34 Oct 25 05:02 Desktop
drwxr-xr-x.  2 zcl zcl    6 Oct  5 00:07 Documents
drwxr-xr-x.  2 zcl zcl    6 Oct  5 00:07 Downloads
-rw-rw-r--.  1 zcl zcl  827 Oct  5 00:12 install.sh
-rw-rw-r--.  1 zcl zcl   60 Oct 11 06:46 makefile
[zcl@localhost ~]$ stat Code
  File: ‘Code’
  Size: 4096      	Blocks: 8          IO Block: 4096   directory
Device: fd00h/64768d	Inode: 3270049     Links: 28
Access: (0775/drwxrwxr-x)  Uid: ( 1000/     zcl)   Gid: ( 1000/     zcl)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2018-10-25 22:29:03.493269180 -0400
Modify: 2018-10-24 21:56:01.089329792 -0400
Change: 2018-10-24 21:56:01.089329792 -0400
 Birth: -
```
Access：最后访问时间
Modif：文件内容最后修改时间
Change：属性最后修改时间

## inode
![](https://s2.ax1x.com/2019/05/07/EsQpW9.png)
所以，新建一个文件的主要操作是：

1. 存储属性
内核首先找到一个空闲的inode，在这里是34192477，内核把文件信息记录到其中
2. 存储数据
该文件存储在三个磁盘块，内核找到了三个空闲块，29，57，1228，将内核缓冲区数据复制到29，下一块复制到57，以此类推
3. 记录分配情况
文件按顺序29，57，1228存放，内核在inode上的磁盘分布区记录了上述块列表
4. 添加文件名到目录
新的文件名叫做myfile。内核将入口（34192477，myfile）添加到目录文件，文件名核inode之间的对应关系将文件名和文件的内容及属性连接起来
## 硬链接和软链接
上面我们了解了inode，是不是每个文件都有自己的独立的inode 呢？也不一定
### 硬链接
**在Linux上可以将多个文件名对应同一个inode**，那么这个就是硬链接
```bash
[tim@xpu ~]$ touch myfile
[tim@xpu ~]$ ln myfile mylink
[tim@xpu ~]$ ls -li myfile mylink
1067027 -rw-rw-r-- 2 tim tim 0 Oct 26 17:38 myfile
1067027 -rw-rw-r-- 2 tim tim 0 Oct 26 17:38 mylink
[tim@xpu ~]$ ll
total 16
drwxrwxr-x 2 tim tim 4096 Oct 16 18:32 11_code
drwxrwxr-x 2 tim tim 4096 Oct 16 18:32 12_code
drwxrwxr-x 2 tim tim 4096 Oct  1 22:09 lhl
-rw-rw-r-- 2 tim tim    0 Oct 26 17:38 myfile
-rw-rw-r-- 2 tim tim    0 Oct 26 17:38 mylink
drwxrwxr-x 3 tim tim 4096 Oct  2 18:41 zcl
[tim@xpu ~]$
```
我们可以看到myfile和mylink的inode号是一样的，那么这就属于硬链接，所以myfile与mylink共用一个inode，所以所对应的物理设备也是只有一份文件，同样的我们可以看出来myfile和mylink的硬链接数为2，接下来说说目录文件的硬链接数目，每个目录中的子目录都有 `.` 和 `..` ，`.` 就表示当前目录， `..` 就表示上一级目录，所以一个空目录都包含两个硬链接数，如果包含子目录的话那么硬链接数还应该加上子目录的个数，因为子目录中中的每个目录都含有一个 `..` 与父目录硬链接，使用`ln`命令实现文件之间的硬链接，使用方法在上述代码中已经包含！
### 软链接
使用`ln -s`选项可以建立软链接，软链接有自己独立的inode，软链接保存了其代表的文件的绝对路径，是另外一种文件，在硬盘上有独立的区块，访问时替换自身路径。所以可以把软链接看成是Windows底下的快捷方式!
```bash
[tim@xpu code]$ touch myfile
[tim@xpu code]$ ln myfile mylink
[tim@xpu code]$ ll
total 0
-rw-rw-r-- 2 tim tim 0 Oct 26 18:00 myfile
-rw-rw-r-- 2 tim tim 0 Oct 26 18:00 mylink
[tim@xpu code]$ ln -s myfile _mylink                                                                                                                                           
[tim@xpu code]$ ls -li
total 0
1067046 -rw-rw-r-- 2 tim tim 0 Oct 26 18:00 myfile
1067046 -rw-rw-r-- 2 tim tim 0 Oct 26 18:00 mylink
1067047 lrwxrwxrwx 1 tim tim 6 Oct 26 18:01 _mylink -> myfile
```
接下来加入我们删除myfile，硬链接mylink是正常的，但是软链接却报警告：
![](https://s2.ax1x.com/2019/05/07/EsQEdO.png)
不难理解， 对于硬链接只要还存在硬链接那么即使删除其中一个，那么inode也不会释放，那么磁盘数据也不会释放，对于软链接来说，软链接由于是一个独立的文件保存了其指向文件的路径，所以只要myfile被删除，那么路径也就没有意义了！