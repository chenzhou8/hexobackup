---
title: Java中数组复制的效率比较
date: 2018-02-28 10:41:13
toc: true
categories: Java核心技术
---

在开发中，数组复制是经常使用的，很多方法都可以进行数组赋值，但是效率却天差地别：效率最高的是：`System.arraycopy()`, 下面是它的使用方式的参数说明：

![](https://s2.ax1x.com/2019/05/01/EYBlHU.png)

我们可以看看它的源代码，它是个native方法，毫无疑问效率最高：

![](https://s2.ax1x.com/2019/05/01/EYBG4J.png)

再说说Arrays.copyof()方法，看源代码发现，它还是调用了System.arraycopy()方法：

![](https://s2.ax1x.com/2019/05/01/EYBdu6.png)

然后呢，再看看Object类的clone方法：

![](https://s2.ax1x.com/2019/05/01/EYBxVU.png)

clone()的返回值是Object类型，强制类型转换毫无疑问是降低了效率，但是好歹是native方法，不会存在有特别明明显的差距的。当然自己通过for循环的方式也可以进行数组的复制，但是效率依旧是很低的！所以还是推荐用`System.arraycopy()` 来进行数组的复制吧！