---
title: JVM内存配置参数说明
date: 2018-03-01 09:24:40
toc: true
categories: Java核心技术
---

# JVM内存划分

Xms -Xmx分别设置堆的最小值和最大值，如果要设置成堆的大小可变，那么可以将最大值和最小值设置成不一样，如果要将堆大小固定，那么只需将最大值和最小值设置成一样的就行。
jvm中分为堆和方法区，堆又进一步分为新生代和老年代。方法区为永久代，堆中区分的新生代和老年代是为了垃圾回收，新生代中的对象存活期一般不长，而老年代中的对象存活期较长，所以当垃圾回收器回收内存时，新生代中垃圾回收效果较好，会回收大量的内存，而老年代中回收效果较差，内存回收不会太多。

基于以上特性，新生代中一般采用复制算法，因为存活下来的对象是少数，所需要复制的对象少，而老年代对象存活多，不适合采用复制算法，一般是标记整理和标记清除算法。因为复制算法需要留出一块单独的内存空间来以备垃圾回收时复制对象使用，所以将新生代分为eden区和两个survivor区，每次使用eden和一个survivor区，另一个survivor作为备用的对象复制内存区。

![](https://s2.ax1x.com/2019/05/01/EYraTO.png)

所谓的 Copying算法 是空间换时间，而 Mark-Compact算法 则是时间换空间。因为年轻代中的对象基本都是朝生夕死的(80%以上)，所以在年轻代的垃圾回收算法使用的是复制算法（ Copying算法 ）。
Copying算法： 在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。
Mark-Compact算法： 在工作的时候则需要分别的mark与compact阶段，mark阶段用来发现并标记所有活的对象，然后compact阶段才移动对象，清楚未标记对象来达到compact的目的。如果compact方式是sliding compaction，则在mark之后就可以按顺序一个个对象“滑动”到空间的某一侧。因为已经先遍历了整个空间里的对象图，知道所有的活对象了，所以移动的时候就可以在同一个空间内而不需要多一份空间。



# 常见配置汇总

* 堆设置
    * Xms:初始堆大小
    * Xmx:最大堆大小
    * XX:NewSize=n:设置年轻代大小
    * XX:NewRatio=n:设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
    * XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
    * XX:MaxPermSize=n:设置持久代大小
* 收集器设置
    * XX:+UseSerialGC:设置串行收集器
    * XX:+UseParallelGC:设置并行收集器
    * XX:+UseParalledlOldGC:设置并行年老代收集器
    * XX:+UseConcMarkSweepGC:设置并发收集器
* 垃圾回收统计信息
    * XX:+PrintGC
    * XX:+PrintGCDetails
    * XX:+PrintGCTimeStamps
    * Xloggc:filename
* 并行收集器设置
    * XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数。并行收集线程数。
    * XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间
    * XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
* 并发收集器设置
    * XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。
    * XX:ParallelGCThreads=n:设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。