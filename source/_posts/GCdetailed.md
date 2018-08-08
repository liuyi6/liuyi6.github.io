---
title: 垃圾收集器详解
date: 2018-08-7 23:13:19
tags: [垃圾收集]
categories: jvm
---

前言
---
垃圾收集器作为内存回收的具体表现，Java虚拟机规范并未对垃圾收集器的实现做规定，因而不同版本的虚拟机有很大产别，因而我们在这里主要讨论基于Sun HotSpot虚拟机1.6版本Update22,此虚拟机包含的收集器如下所示：
![](/images/2018-7-31/jvm_GC_Hotspot6.png)
如图展示了7种作用于不同分代的收集器，若两个收集器之间存在连线，说明他们可以搭配使用。我们堆收集器进行比较就是为了针对具体的情况选择最合适的收集器。
Serial收集器
---
Serial是最基本，最早的收集器，曾是JDK1.3.1之前的虚拟机新生代唯一选择，这个收集器是单线程收集器，它不仅仅只会使用一个单线程或一条收集线程区完成垃圾收集工作，更重要的是当进行垃圾收集时，其他工作线程必须暂停，直到收集结束。下图为Serial/Serial Old收集器运行过程：
![](/images/2018-7-31/jvm_GC_Hotspot_serial.png)
从JDK1.3到JDK1.7,HoSpot虚拟机开发团队一直在为消除或减少工作线程因内存回收而导致停顿的现象而努力着，从Serial收集器，到Parallel收集器，再到Concurrent Mark Sweep(CMS)用户线程停顿时间不断缩减，但此现象并未完全消除。
到目前为止，Serial是虚拟机运行在Client模式下的默认的新生代收集器，它也有很多优点：简单高效(与其他收集器单线程相比)，对于限定单个CPU，由于没有线程交互开销，垃圾收集获得最高的单线程收集效率。Serial收集器是运行在Client模式下的虚拟机的好的选择。
ParNew收集器
---
ParNew收集器是Serial收集器的多线程版本，除使用多线程进行垃圾收集外，其余行为包括扣Serial收集器可用的控制参数、收集算法、Stop The World 、对象分配规则、回收策略等与Serial完全一样。ParNew收集器的工作过程如下图所示：
![](/images/2018-7-31/jvm_GC_Hotspot_parnew.png)
ParNew是运行在Server模式下虚拟机种的首选新生代收集器，因为除了Serial收集器，目前只有它能够与CMS收集器配合工作。ParNew收集器由于存在线程交互，可能不会有比Serial更好的效果但随着CPU数量的增加，它对于GC时系统资源的利用有很大好处。
此处区分以下并行和并发：
并行(Parallel):多条垃圾收集线程并行工作，此时用户线程仍处于等待状态。
并发(Concurrent):指用户线程与垃圾收集线程同时执行(不一定并行，可能交替执行)，用户程序继续运行，垃圾收集程序运行在另一个CPU上。
Parallel Scavenge收集器
---
Parallel Scavenge收集器是一个新生代收集器，它时复制算法的收集器，也是并行的多线程收集器，它的关注点与其他收集器不同,其它收集器尽可能缩短用户线程停顿时间，Parallel Scavenge收集器则是达到一个可控制的吞吐量(吞吐量=运行用户代码时间/(运行代码时间+垃圾收集时间))。停顿时间越短越适合需要用户交互程序，良好相应速度能提升用户体验；高吞吐量则可以最高效率的利用CPU时间，尽快完成程序任务，适用于后台运算而不需要太多交互的任务。
Serial Old收集器
---
Serial Old是Serial的老年版本，它也是单线程收集器，使用“标记-整理“算法。它主要是被Client模式下的虚拟机使用。在Server模式下，它有两个用途：一是在JDK1.5及之前的本本种与Parallel Scavenge收集器搭配使用；二是作为CMS收集器的后备份预案，在并发收集发生Corrent Model Failure时候使用。Serial Old的工作过程如下图所示：
![](/images/2018-7-31/jvm_GC_Hotspot_serialold.png)
Parallel Old收集器
---
Parallel Old使用多线程和“标记-整理”算法，于JDK1.6开始提供，是Parallel Scavenge收集器的老年代版本。在其出现前，老年代除了Serial old外别无选择，由于Serial old的“拖累”，使用Parallel Scavenge未必能够在整体上实现吞吐量最大化的效果。Parallel Old出现后，“吞吐量”的黄金组合出现了，在注重吞吐量及CPU资源的敏感场合，优先考虑Parallel Old与Parallel Scavenge收集器组合。Parallel Old工作过程如图 所示：
![](/images/2018-7-31/jvm_GC_Parallel Old.png)
CMS收集器
---
CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。CMS收集器应用于那些重视服务响应速度，系统停顿短的场景。CMS应用了“标记-清除”算法，他的运作过程包括四个场景：

1. 初始标记(CMS initial mark)
2. 并发标记(CMS concurrent mark)
2. 重新标记(CMS ermark)
2. 并发清除(CMS concurrent sweep)

由于在整个过程中耗时最长的并发标记与并发清除过程收集器线程可与用户线程一起工作，因而，可看做并发标记与并发清除的收集器线程是与用户线程一起并发执行的，其运行过程如下图所示：
![](/images/2018-7-31/jvm_GC_CMS.png)
CMS是一款优秀的收集器，他的主要优点有：并发收集，低停顿，因而也称为并发低停顿收集器，但其仍有缺点，主要表现在三个方面：

1. CMS收集器对CPU资源非常敏感。面向并发设计的程序都对CPU比较敏感，在并发阶段，它会占用一部分线程导致应用程序变慢，降低总吞吐量。CMS默认自动回收线程数为(cpu数+3)/4，故当cpu总数小于4 时，几乎要拿出50%的资源执行线程收集器。增量式并发收集器(i-CMS)就是为解决这种情况而创造的，他的思想是：并发标记和并发清理时，让GC线程、用户线程交替运行，尽量减少GC独占资源时间，虽然使得垃圾回收时间变长，但降低了对用户程序的影响。目前，i-CMS已经不再提倡用户使用。
2. CMS收集器无法处理浮动垃圾。当出现“Concurrent Mode Failure”导致另一次Full GC的产生。此时，CMS并发清理阶段用户线程还在运行着，无法集中处理运行程序产生的垃圾。由于垃圾收集阶段用户线程还在运行，需给线程预留内存，要是预留的内存无法满足需求，就会出现“Concurrent Mode Failure”失败，此时，临时启用Serial Old进行老年代垃圾回收，耗费时间过长。
3. CMS算法是基于“标记-清除”的收集器，收集借书产生大量的空间碎片，当无法找到足够的连续空间存放对象就会触动Full GC。未解决这个问题，CMS增加一个碎片整理时间，此过程无法并发，停顿时间变长。

G1收集器
---
G1收集器与CMS相比有两点改进：一是G1是基于“标记-整理”的收集器，不会产生空间碎片。二是他可以精确地控制停顿，能让使用者指定在一个时间片段内消耗在垃圾收集上的时间。G1可以在不牺牲吞吐量的前提下完成低停顿 内存回收。G1将java堆划分为多个大小独立的区域，并根据区域的垃圾堆积程度，维护一个优先列表，每次根据收集时间，优先回收垃圾最多的区域，保证来G1收集器在有效时间内的的回收效率。
垃圾收集器参数汇总
---
参数|描述
---|---
UseSerialGC|虚拟机运行在Client模式下的默认值，打开此开关，使用Serial+Serial Old收集器组合进行内存回收
UseParNewGC |	打开此开关后，使用ParNew+Serial Old的收集器组合进行内存回收
UseConcMarkSweepGC |	打开此开关后，使用ParNew+CMS+Serial Old的收集器组合进行内存回收。Serial Old收集器将作为CMS收集器出现Concurrent Mode Failure失败后的后备收集器使用
UseParallelGC |	虚拟机运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge + Serial Old（PS MarkSweep）的收集器组合进行内存回收
UseParallelOldGC 	|打开此开关后，使用Parallel Scavenge + Parallel Old的收集器组合进行内存回收
SurvivorRatio 	|新生代中Eden区域与Survivor区域的容量比值，默认值为8，代表Eden：Survivor=8：1
PretenureSizeThreshold |	直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配
MaxTenuringThreshold 	|晋升到老年代的对象年龄，每个对象在坚持过一次Minor GC之后，年龄就增加1，当超过这个参数时就进入老年代
UseAdaptiveSizePolicy |	动态调整Java堆中各个区域的大小以及进入老年代的年龄
HandlePromotionFailure |	是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden和Survivor区的所有对象都存活的极端情况
ParallelGCThreads |	设置并行GC时进行内存回收的线程数
GCTimeRatio 	|GC时间占总时间的比率，默认值为99，即允许1%的GC时间。仅在使用Parallel Scavenge收集器时生效
MaxGCPauseMillis |	设置GC的最大停顿时间，仅在使用Parallel Scavenge收集器时生效
CMSInitingOccupancyFraction |	设置CMS收集器在老年代空间被使用多少后触发垃圾收集。默认值为68%，仅在使用CMS收集器时生效
UseCMSCompactAtFullCollection|设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片整理，仅在使用CMS收集器时生效
CMSFullGCsBeforeCompaction|设置CMS收集器在进行若干次垃圾收集后再启动一次内存碎片整理。仅在使用CMS收集器时生效

本文主要参考自《深入理解Java虚拟机——JVM高级特性与最佳实践》一书