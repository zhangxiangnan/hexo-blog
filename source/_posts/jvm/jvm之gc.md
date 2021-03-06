---
title: jvm之gc
date: 2016-07-02 11:23:01
tags:
- jvm
- gc
categories: jvm
---


jvm之gc知识

<!-- more -->

###### 1、gc如何判定对象是否为垃圾，即如何判定哪些对象还活着，哪些已然死去？    
　　1、引用计数算法：   
　　　有个地方引用某对象，引用加一，引用失效，引用减一，引用为0则该对象为垃圾。其无法解决相互循环引用的问题。主流jvm虚拟机没有采取该算法。   
　　2、可达性分析算法   
　　从一系列gc roots的对象作为起点，向下搜索，gc roots能通过引用关系到达某个对象，则该对象有效，否则垃圾。   

###### 2、哪些对象可以作为gc roots    
```
-  虚拟机栈（栈帧中的本地变量表）中引用的对象   
-  方法去中类静态属性引用的对象
-  方法去中常亮引用的对象
-  本地方法栈中JNI（native方法）引用的对象
```
###### 3、垃圾收集算法原理、特点、用处。
　　1、标记-清除（清理）算法   
  　　mark-sweep，首先标记所有需要回收的对象，标记完成后统一回收。   
　　不足：   
　　　　- 效率问题，标记、清除过程效率不高；   
　　　　- 空间问题，标记清除后产生大量不连续内存碎片，碎片多在分配大对象使时，无法找到连续内存，而触发另一次垃圾收集。

　　2、复制算法（标记-复制，常用于新生代）  
　　思想是将内存容量分成大小相等2块，每次使用其中一块，一块用完了，将存活对象复制到另一块，然后把已使用的一次清理。    
　　现在商业虚拟机采取这种算法回收新生代，也不是对半分内存，而是分一块较大的Eden伊甸园空，和2块较小的Survivor存活空间，每次使用Eden和一块Survivor空间。回收时，将Eden和Survivor存活的拷贝到另一个Survivor空间（不够有老年代的担保分配机制），然后清理Eden和使用的一个Survivor区。   
　　HotSpot虚拟机默认Eden:Survivor:Survivor = 8:1:1，浪费10%内存。   
　　算法效率比标记清除高，内存利用率低。

  3、标记整理算法（标记压缩，常用于老年代）    
　　Mark-Compact，与标记清除类似，让所有存活对象都向一端移动，然后直接清理掉边界以外的内存。
　　减少内存碎片，存活对象移动到一端需要时间。


  4、分代收集算法    
　　当前商业虚拟机都采用分代收集（General Collection）算法。   
　　根据对象存货周期将堆内存分新生代，老年代，新生代使用复制算法，老年代使用标记清理或者标记整理算法。

###### 4、minor gc与full gc分别何时触发
　　Minor GC指发生在新生代的垃圾收集操作，java对象大都朝生熄灭，所以Minor GC非常频繁，速度也较快。   
　　Major GC(Full GC)发生在老年代，经常伴有至少一次的Minor GC，full GC速度比minor GC慢10倍以上。   
　　2中gc触发时间：   
```
- 多数情况，对象在新生代Eden区分配，当Eden区没有足够空间分配，则发起一次Minor GC。
- Minor GC 首先会对Eden区的对象进行标记，标记出来存活的对象。然后把存活的对象copy到From空间。（标记-复制）
```
如果From空间足够，则回收eden区可回收的对象。
如果from内存空间不够，则把From空间存活的对象复制到To区，
如果TO区的内存空间也不够的话，则把To区存活的对象复制到老年代。
如果老年代空间也不够（或者达到触发老年年垃圾回收条件的话）则触发一次full GC。
- System.gc()若被调用，触发Full GC。

###### 5、垃圾收集器哪些？CMS与G1收集器的特点
并行：parallel，收集线程与用户线程并行工作，收集线程工作时，用户线程处于等待状态
并发：concurrent，用户线程与收集线程同时运行。

- Serial收集器       
  单线程、串行回收、stop the world
  工作于新生代，采取复制算法
  client模式下的默认收集器，无线程切换开销，速度快，适合单cpu或client用户桌面应用场景（新生代一般配置的内存小，回收快）。

- ParNew收集器   
Serial的多线程版本、Stop the world、并行回收
工作于新生代，采取复制算法
server模式下新生代的首选收集器，CMS只能与Serial和ParNew配合工作。

- Parallel Scavenge收集器    
新生代收集器、采取复制算法，并行多线程
关注吞吐量（用户代码执行/用户代码执行 + 垃圾收集）,高效利用cpu，尽快完成运算任务，适合后台运算而不需要太多交互的任务。
吞吐量优先收集器
-XX：MaxGCPauseMills  -XX:GCTimeRatio
自适应调节策略，来达到最大的吞吐量或最合适的响应时间。

- Serial Old收集器   
serial的老年代版本，单线程，使用标记-整理算法
适合client模式下的虚拟机
stop the world
server模式下，作为JDK1.5以及之前版本与Parallel Scavenge收集器搭配，或者作为CMS收集器的后备。

- Parallel Old收集器   
Parallel Scavenge的老年版本，多线程、使用标记-整理算法
JDK1.6提供。
吞吐量优先及CPU资源敏感场合，优先考虑Parallel Scavenge + Paralle Old收集器。

- CMS收集器：   
concurrent mark sweep 并发标记清除
以获取最短回收停顿时间为目标
非常适合服务的响应速度，希望系统停顿时间最短，给用户带来较好体验。
基于标记-清除算法
分初始标记、并发标记、重新标记、并发清除，前2个过程stop the world。
并发低停顿收集器    
缺点
```
   - 对cpu资源非常敏感
   - 无法处理浮动垃圾
   - 大量空间碎片
```
- G1收集器   
 面向服务端应用
 特点：
```
  - 并行与并发，缩短停顿时间
  - 分代收集，和其他收集器一样的概念
  - 空间整合，整体看是标记-整理
  - 可预测的停顿模型
  - 建立优先列表，优先回收价值最大的区域。
  - 各区域之间通过建立记忆集避免全表扫描引用依赖关系。
```
  总结：
  G1与CMS相比，若立足于低停顿，CMS是现在选择， 以后可能G1更优秀，G1是可尝试选择。
