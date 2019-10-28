---
title: Linux下打包静/动态库
date: 2018-10-29 16:10:05
toc: true
categories: Linux系统编程
---

## 静态库和动态库概念
### 静态库
在Linux下是`.a`的后缀名，在windows下是`.lib`的后缀名，程序在编译链接的时候吧库代码链接到可执行文件中，运行时不再需要静态库
### 动态库
在Linux下是`.so` ，在windows下是`.dll`，程序在运行的时候才去链接动态库的代码，多个程序共享库的代码！
一个与动态库链接的可执行文件仅仅包含它用到的函数的入口地址的一个表，而不是外部函数所在目标文件的整个机器码
在可执行文件运行以前，外部函数的机器码，由操作系统从磁盘上复制到内存中，这个过程就是动态链接
动态库可以在多个程序直接共享，所以使得可执行文件变得更小，节省磁盘空间，操作系统采用虚拟内存机制允许物理内存中的一份动态库被要用到该库的所有进程共用，节省了内存和磁盘空间
## 打包静态库与动态库
### 打包静态库
假设有以下代码，我们需要把add和sub打包成mymath静态库！
![](https://s2.ax1x.com/2019/05/07/EsQaSs.png)

```bash
[zcl@localhost 25_code]$ ls
add.c  add.h  main.c  sub.c  sub.h
[zcl@localhost 25_code]$ gcc -c add.c -o add.o
[zcl@localhost 25_code]$ gcc -c sub.c -o sub.o
[zcl@localhost 25_code]$ ar -rc libmymath.a add.o sub.o
[zcl@localhost 25_code]$ ls
add.c  add.h  add.o  libmymath.a  main.c  sub.c  sub.h  sub.o
[zcl@localhost 25_code]$ ar -tv libmymath.a 
rw-rw-r-- 1000/1000   1240 Oct 28 03:58 2018 add.o
rw-rw-r-- 1000/1000   1240 Oct 28 03:58 2018 sub.o
[zcl@localhost 25_code]$ gcc main.c -L. -lmymath
[zcl@localhost 25_code]$ ls
add.c  add.h  add.o  a.out  libmymath.a  main.c  sub.c  sub.h  sub.o
[zcl@localhost 25_code]$ ./a.out 
add_ret = 5
sub_ret = 3
[zcl@localhost 25_code]$
```
![](https://s2.ax1x.com/2019/05/07/EsQrwT.png)
#### 库搜索路径
* 从左到右搜索`-L`指定的目录
* 由环境变量指定的目录（LIBRARY_PATH）
* 由系统指定的目录 `/usr/lib`、` /usr/local/lib`
### 打包动态库
```bash
[zcl@localhost 25_code]$ ls
add.c  add.h  libmymath.a  main.c  sub.c  sub.h
[zcl@localhost 25_code]$ gcc -fPIC -c add.c sub.c
[zcl@localhost 25_code]$ gcc -shared -o libmymath.so add.o sub.o
[zcl@localhost 25_code]$ ls
add.c  add.h  add.o  libmymath.a  libmymath.so  main.c  sub.c  sub.h  sub.o
[zcl@localhost 25_code]$ gcc -c main.c -o main.o
[zcl@localhost 25_code]$ gcc main.o -o a2.out -L. -lmymath
[zcl@localhost 25_code]$ ls
a2.out  add.c  add.h  add.o  libmymath.a  libmymath.so  main.c  main.o  sub.c  sub.h  sub.o
[zcl@localhost 25_code]$ export LD_LIBRARY_PATH=.
[zcl@localhost 25_code]$ ./a2.out 
add_ret = 5
sub_ret = 3
```
![](https://s2.ax1x.com/2019/05/07/EsQsTU.png)
其它的添加动态库方式：
1、往/lib和/usr/lib里面加东西，是不用修改/etc/ld.so.conf文件的，但是添加完后需要调用下ldconfig，不然添加的library会找不到。
2、如果添加的library不在/lib和/usr/lib里面的话，就一定要修改/etc/ld.so.conf文件，往该文件追加library所在的路径，然后也需要重新调用下ldconfig命令。
3、如果添加的library不在/lib或/usr/lib下，但是却没有权限操作写/etc/ld.so.conf文件的话，这时就需要往export里写一个全局变量LD_LIBRARY_PATH，就可以了。
参考文章：
[《Linux动态链接库的使用》](https://www.cnblogs.com/52php/p/5681755.html)
[《linux下动态链接库》](https://www.cnblogs.com/Anker/p/3527677.html)