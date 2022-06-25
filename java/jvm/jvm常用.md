[toc]



# 虚拟机的概念



所以其实`Java虚拟机，就是一个字节码翻译器`，它将字节码文件，翻译成各个系统对应的机器码，确保字节码文件能够在各个操作系统正确运行。

![img](https://img2018.cnblogs.com/blog/595137/201812/595137-20181212194650912-14632118.png)



# 编译器



## javac（源代码 => 字节码）

JDK的安装目录中，有一个`javac`工具，用于将Java代码翻译成字节码，这个工具我们叫做`编译器`，相对于后面要说的其他编译器，因为处于编译的前期，所以又被称为`前端编译器`。

![img](https://img2018.cnblogs.com/blog/595137/201812/595137-20181212194443807-900872025.png)



## JIT（字节码 => 机器码）

Just-In-Time，即时编译器。

将字节码转化为本地机器代码。



# Java虚拟机内存结构



## 公有部分：Java堆，方法区，常量池

在Java虚拟机中，线程共享部分包括Java堆，方法区，常量池。



 ### Java堆

用于Java实例对象的内存分配，几乎所有实例对象都会在这里进行内存的分配。

Java堆根据对象存活时间的不同，被分为年轻代，老年代两个区域，年轻代被进一步分为Eden区，From Survivor 0，To Survivor 1区

![img](https://img2018.cnblogs.com/blog/595137/201901/595137-20190103103329413-247778313.png)



- 年轻代：

  - Eden区：
    - 对象需要分配时，对象优先被分配在Eden区
    - Eden区域内存不够时，Java虚拟机会启动垃圾回收，此时Eden区中没有被引用的对象的内存就会被回收，其他没有被回收的对象被放入到空的survivor区，一些存活时间较长的对象会进入到老年代

  - survivor
    - 用于保存在eden space[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)区域中经过垃圾回收后没有被回收的对象
    - Survivor有两个，分别为To Survivor、 From Survivor，这个两个区域的空间大小是一样的
    - 执行垃圾回收的时候Eden区域不能被回收的对象被放入到空的survivor（也就是To Survivor，同时Eden区域的内存会在垃圾回收的过程中全部释放），另一个survivor（即From Survivor）里不能被回收的对象也会被放入这个survivor（即To Survivor），然后To Survivor 和 From Survivor的标记会互换，始终保证一个survivor是空的
    - 新生代中执行的垃圾回收被称之为Minor GC（因为是对新生代进行垃圾回收，所以又被称为Young GC），每一次Young GC后留下来的对象age加1。

- 老年代：

  - 用于存放新生代中经过多次垃圾回收仍然存活的对象，也有可能是新生代分配不了内存的大对象会直接进入老年代。
  - 当老年代被放满的之后，虚拟机会进行垃圾回收，称之为Major GC，又叫Old GC。
  - 由于Major GC除并发GC外均需对整个堆进行扫描和回收，因此又称为Full GC。



默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制，可以通过MinHeapFreeRatio参数进行调整；

默认空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制，可以通过MaxHeapFreeRatio参数进行调整。



### 方法区

存储了每一个类的结构信息，如运行时常量池，字段，方法数据，构造方法等等。



- JDK 1.7中，方法区被称为永久代，Permanent Space，永远不会被JVM垃圾回收。
- JDK 1.8中，方法区被称为MetaSpace



## 私有部分：每个线程的私有数据

线程私有部分：PC寄存器，Java虚拟机栈，本地方法栈

### PC寄存器

Program Counter寄存器，保存线程当前正在执行的方法。

### Java虚拟机栈

栈与线程同时创建，用于存储栈帧，即存储局部变量与一些过程结果。





# 垃圾回收



## G1回收器

G1回收器是JDK1.7中使用的全新垃圾回收器。



分代来看，G1属于分代垃圾回收器，但是他使用了分区算法，使得Eden区，From区，Survivor区和老年代等各种内存不必连续。



G1回收器，将其一大块的内存分为许多细小的区块，所以不要求内存是连续的。

每一块都是一个Region。

![img](https://img2018.cnblogs.com/blog/595137/201901/595137-20190103110304049-1810895125.png)



H：Humongous，表示这些 Region 存储的是巨型对象（humongous object，H-obj）。当新建对象大小超过 Region 大小一半时，直接在新的一个或多个连续 Region 中分配，并标记为 H。



## 垃圾回收类型



### Minor GC

从年轻代空间回收内存，称为Minor GC，也叫做Young GC。

- 当JVM无法为一个新的对象分配空间时，触发Minor GC
- Eden区满了，会触发Minor GC，所以Eden区越小，越频繁执行Minor GC
- Survivor满不会引发GC
- 当年轻代中的Eden区分配满时，年轻代中的部分对象会晋升到老年代，所以Minor GC后，老年代的占用量通常会有所升高
- 对于大部分应用程序，停顿导致的延迟都是可以忽略不计的，因为大部分 Eden 区中的对象都能被认为是垃圾，永远也不会被复制到 Survivor 区或者老年代空间。



### Major GC

从老年代空间回收内存，被称为Major GC，也被称为Old GC。

许多Major GC是由Minor GC触发的。



1. 分配对象内存时发现内存不够，触发 Minor GC
2. Minor GC 会将对象移到老年代中
3. 如果此时老年代空间不够，那么触发 Major GC



### Full GC

Full GC是清理整个堆空间，包括年轻代，老年代，永久代。

- 当要触发Minor GC时，如果发现young GC的平均晋升大小比目前old gen剩余的空间大，不会触发Minor GC，会触发Full GC
- 在永久代分配空间，但已经没有足够空间时，也会触发Full GC



### Stop-The-World

指在进行垃圾回收时因为标记或清理的需要，必须让所有执行任务的线程停止执行任务，从而让垃圾回收线程回收垃圾的时间间隔。

在 Stop-The-World 这段时间里，所有非垃圾回收线程都无法工作，都暂停下来。只有等到垃圾回收线程工作完成才可以继续工作。

Stop-The-World 时间的长短将关系到应用程序的响应时间，因此在 GC 过程中，Stop-The-World 的时间是一个非常重要的指标。



# JVM参数

|         参数         |                   含义                    | 备注                                                         |
| :------------------: | :---------------------------------------: | ------------------------------------------------------------ |
|         -Xms         |                初始堆大小                 | 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制 |
|         -Xmx         |                最大堆空间                 | 默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制 |
|         -Xmn         |              设置新生代大小               |                                                              |
|  -XX:SurvivorRatio   | 设置新生代eden空间和from/to空间的比例关系 |                                                              |
|     -XX:PermSize     | 方法区初始大小，永久代初始大小（JDK1.7）  |                                                              |
|   -XX:MaxPermSize    | 方法区最大大小，永久代最大大小（JDK1.7）  |                                                              |
|  -XX:MetaspaceSize   |          元空间GC阈值（JDK1.8）           |                                                              |
| -XX:MaxMetaspaceSize |         最大元空间大小（JDK1.8）          |                                                              |
|         -Xss         |            每个线程的堆栈大小             |                                                              |



# GC日志配置

### 

| 参数                                  | 含义                                             |
| :------------------------------------ | :----------------------------------------------- |
| -XX:PrintGC                           | 打印GC日志                                       |
| -XX:+PrintGCDetails                   | 打印详细的GC日志。还会在退出前打印堆的详细信息。 |
| -XX:+PrintHeapAtGC                    | 每次GC前后打印堆信息。                           |
| -XX:+PrintGCTimeStamps                | 打印GC发生的时间。                               |
| -XX:+PrintGCApplicationConcurrentTime | 打印应用程序的执行时间                           |
| -XX:+PrintGCApplicationStoppedTime    | 打印应用由于GC而产生的停顿时间                   |
| -XX:+PrintReferenceGC                 | 跟踪软引用、弱引用、虚引用和Finallize队列。      |
| -XLoggc                               | 将GC日志以文件形式输出。                         |





