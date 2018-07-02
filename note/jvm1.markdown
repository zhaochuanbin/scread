---
layout : post
title : "深入JVM之内存模型和GC"
category : java
tagline: ""
date : 2016-04-11
tags : [java,JVM]
---

![](../images/jvm1.jpg)

名称 | 特性 | 作用 | 配置参数 | 异常
-----|------|-----|-----|-----
程序计数器|占用内存小,线程私有,生命周期与线程相同|大致为字节码行号指示器|无|无
虚拟机栈|线程私有,生命周期与线程相同,使用连续的内存空间 | java方法执行的内存模型,存储局部变量表、操作栈、动态链接、方法出口等信息       | -Xss | StackOverflowError、OutOfMemoryError
java堆 | 线程共享,生命周期和虚拟机相同,可以不适用连续的内存地址 | 保存对象实例,所有对象实例(包括数组)都要在堆上分配 | -Xmx，  -Xms ， -Xmn -XX:NewRatio=4  -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=0 | OutofMemoryError
方法区 | 线程共享,生命周期与虚拟机相同,可以使用不连续的内存地址 | 存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据 | -XX: PermSize: 16M, -XX: MaxPermSize: 64M | OutofMemoryError
运行时常量池 | 方法区的一部分,具有动态性 | 存放字面量和符号引用 | |OutOfMemoryError

### 直接内存
直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现。
在JDK 1.4 中新加入了NIO（NewInput/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O 方式，它可以使用Native 函数库直接分配堆外内存，然后通过一个存储在Java 堆里面的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java 堆和Native 堆中来回复制数据。

### 对象是否回收
- ** 引用计数算法:给对象添加一个引用计数器,每当引用的地方,计数器就加1,当引用失效时,计数器就减一,计数器为0时,对象就不可再被引用.**

   缺点:无法解决对象间相互循环引用.

- **可达性分析算法:以GC Roots对象为起点,从这些节点开始向下搜索,搜索所走过的路径就是引用链,当一个对象到GC Roots没有任何引用链相连,则对象是可被回收的.**

### 垃圾收集算法
- ** 标记清除算法**
- ** 标记复制算法**
- ** 标记整理算法**
- ** 分代收集算法**

### g1收集器特点
- ** 并行与并发**
- ** 分代收集**
- ** 空间整合**
- ** 可预测的停顿**

### 内存分配与回收策略
- ** 对象优先在eden区**
- ** 大对象直接进老年代**
- ** 长期存活的对象进入老年代**
- ** 动态年龄判定**
- ** 空间分配担保**

GC回收器
串行收集器:适合小数据量的数据。
并行收集器:吞吐量优先.适用于科学技术和后台处理等
- jVM -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20
-XX:+UseParallelGC：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
-XX:ParallelGCThreads=20，配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。

- JVM -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
-XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
- JVM -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC  -XX:MaxGCPauseMillis=100
-XX:MaxGCPauseMillis=100:设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。

- JVM -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC  -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

并发收集器响应时间优先。适用于应用服务和电信领域。
- JVM -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
-XX:+UseConcMarkSweepGC：设置年老代为并发收集。测试中配置这个以后，-XX:+NewRatio=4 的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
-XX:+UseParNewGC:设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
- JVM -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:+CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
-XX:+CMSFullGCsBeforeCompaction：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
-XX:+UseCMSCompactAtFullCollection：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

JVM辅助参数
-XX:+PrintGC 打印GC信息
-XX:+PrintGCDetails 打印GC详细信息
-XX:+PrintGCTimeStamps -XX:+PrintGC 时间 +GC信息
-XX:+PrintGCApplicationConcurrentTime:打印每次垃圾回收前，程序未中断的执行时间
-XX:+PrintGCApplicationStoppedTime：打印垃圾回收期间程序暂停的时间
-XX:+PrintHeapAtGC 打印GC前后的详细堆栈信息
-Xloggc: filename: 与上面几个配合使用，把相关日志信息记录到文件以便分析。

![](../images/gc.png)

总结：
堆设置
-Xms:初始堆大小
-Xmx:最大堆大小
-XX:NewSize=n :设置年轻代大小
-XX:NewRatio=n: 设置年轻代和年老代的比值
-XX:SurvivorRatio=n: 年轻代中Eden区与两个Survivor区的比值
-XX:MaxPermSize=n: 设置持久代大小
收集器设置
-XX:+UseSerialGC:设置串行收集器
-XX:+UseParallelGC:设置并行收集器
-XX:+UseParalledlOldGC:设置并行年老代收集器
-XX:+UseConcMarkSweepGC:设置并发收集器
垃圾回收统计信息
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:filename
并行收集器设置
-XX:ParallelGCThreads=n: 设置并行收集器收集时使用的CPU数。并行收集线程数。
-XX:MaxGCPauseMillis=n: 设置并行收集最大暂停时间
-XX:GCTimeRatio=n: 设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
并发收集器设置
-XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。
-XX:ParallelGCThreads=n: 设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。

调优
### 年轻代大小设置
- 响应时间优先的应用：尽可能设大，直到接近系统的最低响应时间限制（根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。
- 吞吐量优先的应用：尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。

### 年老代大小设置
响应时间优先的应用：年老代使用并发收集器，所以其大小需要小心设置，一般要考虑并发会话率和会话持续时间等一些参数。如果堆设置小了，可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了，则需要较长的收集时间。
最优化的方案，一般需要参考以下数据获得：
- 并发垃圾收集信息
- 持久代并发收集次数
- 传统GC信息
- 花在年轻代和年老代回收上的时间比例
- 减少年轻代和年老代花费的时间，一般会提高应用的效率

吞吐量优先的应用：一般吞吐量优先的应用都有一个很大的年轻代和一个较小的年老代。原因是，这样可以尽可能回收掉大部分短期对象，减少中期的对象，而年老代尽存放长期存活对象。

### 较小堆引起的碎片问题
因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。当收集器回收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，如果并发收集器找不到足够的空间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。如果出现“碎片”，可能需要进行如下配置：
-XX:+UseCMSCompactAtFullCollection：使用并发收集器时，开启对年老代的压缩。
-XX:CMSFullGCsBeforeCompaction=0： 上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩
