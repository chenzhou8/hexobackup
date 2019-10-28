---
title: 最冤枉的sizeof
date: 2018-04-01 16:56:48
toc: false
categories: C/C++
---

之前一直以为sizeof(char)、sizeof(int)...居然一直以为sizeof是函数，其实sizeof在C中只是一个关键字！！！
之前一直以为sizeof(char)、sizeof(int)...居然一直以为sizeof是函数，其实sizeof在C中只是一个运算符！！！
之前一直以为sizeof(char)、sizeof(int)...居然一直以为sizeof是函数，其实sizeof在C中只是一个操作符！！！

在 Pascal 语言中，sizeof() 是一种内存容量度量函数，功能是返回一个变量或者类型的大小（以字节为单位）
① 在 C 语言中，sizeof() 是一个判断数据类型或者表达式长度的运算符（以字节为单位）

在Pascal 语言与C语言中，对 sizeof() 的处理都是在编译阶段进行;

② sizeof内部的表达式不参与运算

![](https://s2.ax1x.com/2019/05/02/EttxRP.png)

指针记录了另一个对象的地址。既然是来存放地址的，那么它当然等于计算机内部地址总线的宽度。所以在32位计算机中，一个指针变量的返回值必定是4（注意结果是以字节为单位），但是，在64位系统中指针变量的sizeof结果为8

注意sizeof和strlen的区别：
1、strlen(char*)函数求的是字符串的实际长度，直到遇到第一个'\0'，然后就返回计数值，且不包括'\0'。而sizeof()函数返回的是变量声明后所占的内存数，不是实际长度

2、sizeof操作符的结果类型是size_t，它在头文件中typedef为unsigned int类型。该类型保证能容纳实现所建立的最大对象的字节大小

3、sizeof是算符，strlen是函数

4、sizeof可以用类型做参数，strlen只能用char*做参数，且必须是以''\0''结尾的

5、sizeof还可以用函数做参数，比如：short f(); printf("%d\n",sizeof(f())); 结果是sizeof(short)，即2

6、数组做sizeof的参数不退化，传递给strlen就退化为指针了

7、大部分编译程序在编译的时候就把sizeof计算过了是类型或是变量的长度这就是sizeof(x)可以用来定义数组维数的原因

8、strlen的结果要在运行的时候才能计算出来，是用来计算字符串的长度，不是类型占内存的大小

9、sizeof后如果是类型必须加括弧，如果是变量名可以不加括弧。这是因为sizeof是个操作符不是个函数

10、数组作为参数传给函数时传的是指针而不是数组，传递的是数组的首地址