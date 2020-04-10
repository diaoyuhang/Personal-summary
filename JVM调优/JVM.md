# JVM

> ### java从编码到执行

![](D:\0_LeargingSummary\JVM调优\images\java从编码到执行.png)

> ### JDK JRE JVM

![](D:\0_LeargingSummary\JVM调优\images\JDK JRE JVM.png)

# Class File Format

![](D:\0_LeargingSummary\JVM调优\images\class文件格式信息.png)

# Class加载周期

> 1、load加载->赋默认值->初始化值
>
> 2、创建new->申请内存->赋默认值->初始化值

![](D:\0_LeargingSummary\JVM调优\images\class加载过程.png)

# 类加载：loading阶段

类加载器总共有四种：CustomClassLoader,AppClassLoader,ExtensionClassLoader,Bootstrap,

每个加载器加载的内容，都是规定好的（Launcherl类中）

自定向上检查是否已经加载，自定向下进行实际查找和加载

> ### 一个class文件被load到内存中时，分为两部分内容，一部分为二进制内容，另一部分是生成class对象，该对象指向第一部分。
>
> ### 使用final static的变量，不需要加载整个类；

![](D:\0_LeargingSummary\JVM调优\images\类加载器.png)

```java
//ClassLoader 
//finaClass自定义重写，该类用到模版方法
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            //检查该class是否已经被加载过了
            Class<?> c = findLoadedClass(name);
            //没有加载
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    //父类加载器不为空
                    if (parent != null) {
                        //到父加载中查找class
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
				//如果都没有检查到该class
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    //加载class
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

```java
//URLClassLoader
/*
使用从指定资源获得的类字节定义一个类。 生成的Class必须先解析，然后才能使用。
 * Defines a Class using the class bytes obtained from the specified
 * Resource. The resulting Class must be resolved before it can be
 * used.
 */
private Class<?> defineClass(String name, Resource res) throws IOException {
    ...
}
```

> ### 双亲委派，主要出于安全来考虑

如何打破双亲委派机制：

- 如何打破：重写loadClass()

- 何时打破：
- 在jdk1.2之前，自定义ClassLoader都必须重写loadClass
- ThreadContextClassLoader可以实现基础类调用实现类代码，通过thread.setContextClassLoader指定
- 热启动，热部署：tomcat都要自己的模块指定classLoader(可以加载同一类库不同版本)

![](D:\0_LeargingSummary\JVM调优\images\父加载器.png)

# 混合模式

> #### JIT能将java代码编译成本地代码执行

![](D:\0_LeargingSummary\JVM调优\images\混合模式.png)

# Linking阶段

## Verification验证

验证文件是否符合JVM规定

## Preparation

静态成员变量赋默认值

## Resolution

将类、方法、属性等符号引用解析为直接引用

常量池中的各种引用解析为指针、偏移量等内存地址的直接引用

# Initializing阶段

调用类初始化代码，给静态成员变量赋初始值

# JMM(JAVA MEMORY MODEL)java内存模型

## 硬件层数据一致性

intel使用MESI

现代CPU的数据一致性实现=缓存锁（MESI..）+总线锁

读取缓存以cache line为基本单位，目前64bytes

位于同一缓存行的两个不同数据，被两个不同cpu锁定，产生互相影响的为共享问题,一个cpu修改了缓存行需要通知另一个cpu重新读取缓存行

使用缓存行的对齐能够提高效率

## 乱序问题

CPU为了提高指令执行效率，会在一条指令执行过程中（比如去内存读数据（慢100倍）），去同时执行另一条指令，前提是，两条指令没有依赖关系。

## 如何保证特定情况下不乱序

硬件内存屏障 X86

>  sfence:  store| 在sfence指令前的写操作当必须在sfence指令后的写操作前完成。
>  lfence：load | 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。
>  mfence：modify/mix | 在mfence指令前的读写操作当必须在mfence指令后的读写操作前完成。

> 原子指令，如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序

JVM级别如何规范（JSR133）

LoadLoad屏障：
	对于这样的语句Load1; LoadLoad; Load2， 

	在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

StoreStore屏障：

	对于这样的语句Store1; StoreStore; Store2，
	
	在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。

LoadStore屏障：

	对于这样的语句Load1; LoadStore; Store2，
	
	在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。

StoreLoad屏障：

```
对于这样的语句Store1; StoreLoad; Load2，

在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。
```



volatile的实现细节

1. 字节码层面
   ACC_VOLATILE

2. JVM层面
   volatile内存区的读写 都加屏障

   > StoreStoreBarrier
   >
   > volatile 写操作
   >
   > StoreLoadBarrier

   > LoadLoadBarrier
   >
   > volatile 读操作
   >
   > LoadStoreBarrier

3.OS和硬件层面
https://blog.csdn.net/qq_26222859/article/details/52235930
hsdis - HotSpot Dis Assembler
windows lock 指令实现 | MESI实现

synchronized实现细节

1. 字节码层面
   ACC_SYNCHRONIZED
   monitorenter monitorexit
2. JVM层面
   C C++ 调用了操作系统提供的同步机制
3. OS和硬件层面
   X86 : lock cmpxchg / xxx
   [https](https://blog.csdn.net/21aspnet/article/details/88571740)[://blog.csdn.net/21aspnet/article/details/](https://blog.csdn.net/21aspnet/article/details/88571740)[88571740](

## java并发内存模型

每一个线程都有自己的工作内存，从主内存中将变量加载自己的工作空间，然后在返回写到主内存，如果变量加了volatile,则JVM在变量读写前后都加屏障，保证不乱序。

![](D:\0_LeargingSummary\JVM调优\images\java并发内存模型.png)

## 对象的内存布局

- 对象的创建过程

```
class loading
class linking(verification:验证,preparation:静态成员变量赋默认值,resolution:解析)
class initializing:静态成员变量赋初始值
申请内存
成员变量赋默认值
调用构造方法
	成员变量顺序赋初始值
	执行构造方法的语句
```

- 对象在内存中的存储布局

  普通对象

```
对象头：markword 8字节
ClassPointer指针：-XX：+UseCompressedClassPointers为4字节 不开启为8字节
实例数据：1、引用类型：-XX：+UseCompressedOops为4字节 不开启8字节；
Padding对齐（填充对齐），8的倍数
```

​		数组对象

```
1. 对象头：markword 8
2. ClassPointer指针同上
3. 数组长度：4字节
4. 数组数据
5. 对齐 8的倍数
```

- 在内存中占多少个字节

```
JVM默认开启-XX:+UseCompressedClassPointers classPointer指针压缩
-XX:+UseCompressedOops 普通对象指针压缩
Oops= ordinary object pointers 

new Object()占16字节：对象头8字节；ClassPointer指针4字节；没数据；填充对齐4字节
new int[]{} 占16字节，对象头8字节，指针4字节，数组长度4字节；没数据，
new P()占32字节

 private static class P {
                           //8 _markword
                           //4 _oop指针
           int id;         //4
           String name;    //4
           int age;        //4
   
           byte b1;        //1
           byte b2;        //1
   
           Object o;       //4
           byte b3;        //1
   
       }
```

- 对象头具体包括什么

![](D:\0_LeargingSummary\JVM调优\images\对象头信息.png)

> 当一个对象已经计算过identity hashCode,就无法进入偏向锁状态

- 对象怎么定位

```
句柄池 指向了一个间接指针，简介指针分为两部分，一部分指向对象，另一部分指向class
直接指针 直接指向对象所在的内存地址，对象中会指向class
```

# Runtime Data Area

![](D:\0_LeargingSummary\JVM调优\images\Run-time data areas.png)

![](D:\0_LeargingSummary\JVM调优\images\线程共享区域.png)

## pc 程序计数器

> 存放指令位置，一个java线程总是在执行一个方法，这个正在被执行的方法称为当前方法，如果当前方法不是本地方法，PC寄存器就会指向当前正在被执行的指令。如果当前方法是本地方法，那么PC寄存器的值就是undefined
>
> 虚拟机的运行，类似于这样的循环：
>
> while(not end){
>
> ​	取出PC中的位置，找到对应位置的指令;
>
> ​	执行该指令；
>
> PC++；
>
> }

## JVM Stack

![](D:\0_LeargingSummary\JVM调优\images\栈帧.png)

1. Frame-每一个方法对应一个栈帧

```
   1. Local Variable Table局部变量表
   2. Operand Stack操作数栈
      对于long的处理（store and load），多数虚拟机的实现都是原子的
      jls 17.7，没必要加volatile
   3. Dynamic Linking动态链接
      https://blog.csdn.net/qq_41813060/article/details/88379473 
      jvms 2.6.3
   4. return address返回地址
      a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方
```

## Method Area

1.  Perm Space(<1.8)

   字符串常量位于PermSpace

   FGC不会清理

   大小启动的时候指定，不能变

2. Meta Space(>=1.8)

   字符串常量位于堆

   会触发FGC清理

   不设定的话，最大就是物理内存



## Direct Memroy

> JVM可以直接访问的内核空间的内存（OS 管理的内存）
>
> NIO,提高效率，实现零拷贝

## Run_Time Constant Pool

A  run-time constant pool is a pre-class or per-interface run-time representation of the constant_pool table in a class file

运行时常量池是类文件中constant_pool表的前置类或每个接口的运行时表示形式

# GC

## 如何找到垃圾

利用引用计数的方法，来寻找垃圾，当没有引用指向该对象，计数器为0，该对象为垃圾

![](D:\0_LeargingSummary\JVM调优\images\引用计数.png)

根可达算法，解决循环引用的问题

> 什么是根对象：线程启动后，马上需要的对象

![](D:\0_LeargingSummary\JVM调优\images\根可达算法.png)

## GC Algorithms

- Mark-Sweep(标记清除)

> 两边扫描，第一遍找可用标记，第二遍找到不可用并清除，效率低

![](D:\0_LeargingSummary\JVM调优\images\标记清除.png)

![](D:\0_LeargingSummary\JVM调优\images\标记清除示意图.png)



- Copying(拷贝)

![](D:\0_LeargingSummary\JVM调优\images\拷贝算法.png)

![](D:\0_LeargingSummary\JVM调优\images\拷贝算法示意图.png)

- Mark-Compact(标记压缩)

![](D:\0_LeargingSummary\JVM调优\images\标记压缩.png)

![](D:\0_LeargingSummary\JVM调优\images\标记压缩示意图.png)

## 堆内存逻辑分区

![](D:\0_LeargingSummary\JVM调优\images\堆内存逻辑分区.png)

1. 部分垃圾回收器使用的模型

   > 除Epsilon ZGC Shenandoah之外的GC都是使用逻辑分代模型
   >
   > G1是逻辑分代，物理不分代
   >
   > 除此之外不仅逻辑分代，而且物理分代

2. 新生代 + 老年代 + 永久代（1.7）Perm Generation/ 元数据区(1.8) Metaspace

   1. 永久代 元数据 - Class
   2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
   3. 字符串常量 1.7 - 永久代，1.8 - 堆
   4. MethodArea方法区逻辑概念 - 永久代、元数据（存放class原信息，代码编译信息..）

3. 新生代 = Eden + 2个suvivor区 

   1. YGC回收之后，大多数的对象会被回收，活着的进入s0
   2. 再次YGC，活着的对象eden + s0 -> s1
   3. 再次YGC，eden + s1 -> s0
   4. 年龄足够 -> 老年代 （15 CMS 6）
   5. s区装不下 -> 老年代

4. 老年代

   1. 顽固分子
   2. 老年代满了FGC Full GC

5. GC Tuning (Generation)

   1. 尽量减少FGC
   2. MinorGC = YGC
   3. MajorGC = FGC

1. 动态年龄：（不重要）
   

如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代;

   https://www.jianshu.com/p/989d3b06a49d

7. 分配担保：（不重要）
   YGC期间 survivor区空间不够了 空间担保直接进入老年代

   内存分配是在JVM在内存分配的时候，新生代内存不足时，把新生代的存活的对象搬到老生代，然后新生代腾出来的空间用于为分配给最新的对象。这里老生代是担保人。在不同的GC机制下，也就是不同垃圾回收器组合下，担保机制也略有不同。

   参考：https://cloud.tencent.com/developer/article/1082730

![](D:\0_LeargingSummary\JVM调优\images\栈上分配 线程本地分配.png)

## 对象何时进入老年代

> ### 超过 XX:MaxTenuringThreshold 指定次数（YGC）
>
> - Parallel Scavenge 15
> - CMS 6
> - G1 15

> 动态年龄
>
> s1 到 s2超过50%
>
> 把年龄最大的放入old

## 对象分配图

![](D:\0_LeargingSummary\JVM调优\images\总结.png)

![](D:\0_LeargingSummary\JVM调优\images\栈上分配 线程本地分配.png)

> #### 1、全局逃逸（GlobalEscape）
>
> 即一个对象的作用范围逃出了当前方法或者当前线程，有以下几种场景：
>
> - 对象是一个静态变量
> - 对象是一个已经发生逃逸的对象
> - 对象作为当前方法的返回值
>
> #### 2、参数逃逸（ArgEscape）
>
> 即一个对象被作为方法参数传递或者被参数引用，但在调用过程中不会发生全局逃逸，这个状态是通过被调方法的字节码确定的。
>
> #### 3、没有逃逸
>
> 即方法中的对象没有发生逃逸。
>
> **2) 标量替换**
>
> 首先要明白标量和聚合量，基础类型和对象的引用可以理解为标量，它们不能被进一步分解。而能被进一步分解的量就是聚合量，比如：对象。
>
> 对象是聚合量，它又可以被进一步分解成标量，将其成员变量分解为分散的变量，这就叫做标量替换。
>
> 这样，如果一个对象没有发生逃逸，那压根就不用创建它，只会在栈或者寄存器上创建它用到的成员标量，节省了内存空间，也提升了应用程序性能。

## 常见的垃圾回收器

![](D:\0_LeargingSummary\JVM调优\images\常见的垃圾回收器.png)

1. JDK诞生 Serial追随 提高效率，诞生了PS，为了配合CMS，诞生了PN，CMS是1.4版本后期引入，CMS是里程碑式的GC，它开启了并发回收的过程，但是CMS毛病较多，因此目前任何一个JDK版本默认是CMS
   并发垃圾回收是因为无法忍受STW
2. Serial 年轻代 串行回收

![](D:\0_LeargingSummary\JVM调优\images\Serial.png)

1. PS 年轻代 并行回收

![](D:\0_LeargingSummary\JVM调优\images\ParallelScavenge.png)

1. ParNew 年轻代 配合CMS的并行回收![](D:\0_LeargingSummary\JVM调优\images\ParNew.png)

2. SerialOld ![](D:\0_LeargingSummary\JVM调优\images\SerialOld.png)

3. ParallelOld![](D:\0_LeargingSummary\JVM调优\images\ParallelOld.png)

4. ConcurrentMarkSweep 老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)
   CMS问题比较多，所以现在没有一个版本默认是CMS，只能手工指定
   CMS既然是MarkSweep，就一定会有碎片化的问题，碎片到达一定程度，CMS的老年代分配对象分配不下的时候，使用SerialOld 进行老年代回收
   想象一下：
   PS + PO -> 加内存 换垃圾回收器 -> PN + CMS + SerialOld（几个小时 - 几天的STW）
   几十个G的内存，单线程回收 -> G1 + FGC 几十个G -> 上T内存的服务器 ZGC
   
   算法：三色标记 + Incremental Update

![](D:\0_LeargingSummary\JVM调优\images\CMS并发.png)

1. G1(10ms)
   算法：三色标记 + SATB
2. ZGC (1ms) PK C++
   算法：ColoredPointers + LoadBarrier
3. Shenandoah
   算法：ColoredPointers + WriteBarrier
4. Eplison
5. PS 和 PN区别的延伸阅读：
   ▪[https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html#GUID-3D0BB91E-9BFF-4EBB-B523-14493A860E73](https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html)
6. 垃圾收集器跟内存大小的关系
   1. Serial 几十兆
   2. PS 上百兆 - 几个G
   3. CMS - 20G
   4. G1 - 上百G
   5. ZGC - 4T - 16T（JDK13）

1.8默认的垃圾回收：PS + ParallelOld

# JVM调优第一步，了解JVM常用命令行参数

* JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* HotSpot参数分类

  > 标准： - 开头，所有的HotSpot都支持
  >
  > 非标准：-X 开头，特定版本HotSpot支持特定命令
  >
  > 不稳定：-XX 开头，下个版本可能取消

  java -version

  java -X
  
  java -XX:+PrintFlagsFinal打印所有不稳定参数，可通过管道进行筛选匹配

```java
import java.util.List;
import java.util.LinkedList;

public class HelloGC {
  public static void main(String[] args) {
    System.out.println("HelloGC!");
    List list = new LinkedList();
    for(;;) {
      byte[] b = new byte[1024*1024];
      list.add(b);
    }
  }
}
```

1. 区分概念：内存泄漏memory leak，内存溢出out of memory
2. java -XX:PrintCommandLineFlags HelloGC 打印命令行标志

```shell
##初始化堆大小					最大堆大小					
-XX:InitialHeapSize=15934784 -XX:MaxHeapSize=254956544 
#打印命令行标志
-XX:+PrintCommandLineFlags 
# 使用class指针压缩					普通对象指针压缩
-XX:+UseCompressedClassPointers -XX:+UseCompressedOops
```

​	3.java -Xmn 10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags HelloGC

​         PrintGCDetails PrintGCTimeStamps PrintGCCauses

```shell
#初始化堆最小Xms					初始化堆最大Xmx				年轻代大小Xmn
-XX:InitialHeapSize=41943040 -XX:MaxHeapSize=62914560 -XX:MaxNewSize=10485760 
-XX:NewSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGC 
-XX:+UseCompressedClassPointers -XX:+UseCompressedOops 
HelloGC!
[GC (Allocation Failure)  7675K->7424K(39936K), 0.0126873 secs]
[GC (Allocation Failure)  14754K->14591K(39936K), 0.0121782 secs]
[GC (Allocation Failure)  21913K->21760K(39936K), 0.0099799 secs]
[GC (Allocation Failure)  29084K->28928K(39936K), 0.0090766 secs]
[GC (Allocation Failure)  36254K->36096K(45076K), 0.0049278 secs]
[Full GC (Allocation Failure)  36096K->36095K(45076K), 0.0029812 secs]
[GC (Allocation Failure)  43422K->43263K(60416K), 0.0079038 secs]
[GC (Allocation Failure)  50589K->50432K(60416K), 0.0069268 secs]
[Full GC (Allocation Failure)  57758K->57600K(60416K), 0.0070978 secs]
[Full GC (Allocation Failure)  57600K->57589K(60416K), 0.0024856 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at HelloGC.main(HelloGC.java:9)
```

​	4.java -XX:+UseConcMarkSweepGC -XX:+PrintCommandLineFlags -XX:+PrintGC HelloGC

```shell
-XX:InitialHeapSize=15934784 -XX:MaxHeapSize=254956544 -XX:MaxNewSize=84586496 
-XX:MaxTenuringThreshold=6 -XX:OldPLABSize=16 -XX:+PrintCommandLineFlags 
-XX:+PrintGC -XX:+UseCompressedClassPointers -XX:+UseCompressedOops 
-XX:+UseConcMarkSweepGC -XX:+UseParNewGC 
HelloGC!
...
[GC (Allocation Failure)  28001K->27916K(33348K), 0.0029572 secs]
[GC (CMS Initial Mark)  28940K(33348K), 0.0037893 secs]
...
[GC (Allocation Failure)  52579K->52492K(58020K), 0.0066989 secs]
[GC (CMS Final Remark)  53517K(58020K), 0.0016279 secs]
...
[GC (Allocation Failure)  60770K->60684K(92408K), 0.0028012 secs]
[GC (CMS Initial Mark)  64866K(92408K), 0.0012551 secs]
...
[GC (Allocation Failure)  118116K->118030K(123248K), 0.0046752 secs]
[GC (CMS Final Remark)  122213K(123248K), 0.0019949 secs]
```

1. java -XX:+PrintFlagsInitial 默认参数值
2. java -XX:+PrintFlagsFinal 最终参数值
3. java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数
4. java -XX:+PrintFlagsFinal -version |grep GC

# 垃圾回收算法

## 基础概念

-  Card Table 

  如果做YGC时，扫描整个Old区时，效率会非常低，所以JVM设计了CardTable，如果一个old区CardTable中有对象指向Y区，就将它设计为Dirty，下次扫描时，只需要扫描Dirty Card

  在结构上，CardTable使用BitMAP来实现的。

- CSet =Collection Set

  G1：一组可被回收的分区集合。

- RSet = RememberedSet

  G1：记录了其它Region中的对象到本Region的引用；

  RSet的价值在于使得垃圾收集器的不需要扫描整个堆找到谁引用了当前分区中的对象，只需要扫描RSet即可

  ![](D:\0_LeargingSummary\JVM调优\images\RSet.png)

## CMS

## G1

![](D:\0_LeargingSummary\JVM调优\images\G1.png)

- whyG1

分Region回收

优先回收花费时间少，垃圾比例高的Region

- 新老年代的比例

5%~60%，一般不用手工指定，也不要手工指定，因为这是G1预测停顿时间的基准,G1会自动根据STW时间进行调整比例大小

- GC何时触发

  - YGC：Eden空间不足，多线程并行执行
  - FGC:Old空间不足，System.gc()

- G1逻辑分代，物理不分代，G1垃圾回收器会产生FGC

  如果产生FGC：扩充内存；提高CPU性能（回收快，业务逻辑产生对象的速度固定，垃圾回收越快，内存空间越大）；降低MixedGC触发的阈值，让MixedGC提早发生（默认是45%）,老年代占据了堆内存45%的Region

- G1中的MixedGC

  相当于CMS

  XX:InitiatingHeapOccupacyPercent 默认45%

  当超过这个值，启动MixedGC

  - 过程

    初始标记STW

    并发标记

    最终标记STW（重新标记）

    筛选回收STW（并行）

- JAVA10之前是串行FullGC,之后是并行FulllGC

## 并发标记算法：三色标记

- 白色：未被标记的对象
- 灰色：自身被标记，成员变量未被标记
- 黑色：自身和成员变量均已标记完成

1. 漏标

   ![](D:\0_LeargingSummary\JVM调优\images\三色标记_漏标.png)

   解决办法：

   ![](D:\0_LeargingSummary\JVM调优\images\防止漏标.png)

2. 为什么G1用SATB

   灰色指向白色的引用消失时，如果没有黑色指向白色引用，消失的引用会被push到堆栈，下次扫描时拿到这个引用，由于有RSet的存在，不需要扫描整个堆区查找白色的引用，效率比较高。

