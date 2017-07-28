---
title: jvm中的内存分配
date: 2017-07-28
tags: [java, jvm]
categories: 虚拟机
---
## 一、导论
　　java技术体系中所提到的内存自动化管理归根结底就是内存的分配与回收两个问题，之前已经和大家谈过java回收的相关知识，今天来和大家聊聊java对象的在内存中的分配。通俗的讲，对象的内存分配就是在堆上的分配，对象主要分配在新生代的Eden上（关于对象在内存上的分代在垃圾回收中会补上，想了解的也可以参考《深入理解java虚拟机》），如果启动了本地线程分配缓冲，讲按线程优先在TLAB上分配。少数情况下也是直接在老年代中分配。

## 二、经典的分配策略
1、对象优先在Eden上分配

　　一般情况下对象都是优先分配在Eden上，当Eden没有足够的空间进行分配时，jvm会发起一次Minor GC。如果还是没有足够的空间分配，后面还有另外的措施，下面会提到。

　　设置虚拟机的偶记日志参数-XX:+PrintGCDetails，在垃圾回收的时候会打印内存的回收日志，并且在进程退出的时候会输出当前内存各区域的分配情况。下面来看下具体的例子，首先需要设置jvm的参数-Xms20m -Xmx20m -Xmn10m，这三个参数说明java堆大小为20M，且不可扩展，其中10M分配给新生代，剩下的10M分配给老年代。-XX:SurvivorRatio=8是jvm默认的新生代中Eden和Survivor比例，默认为8:1。原因是新生代中的对象98%都会在下一次GC的时候回收掉，所以很适合采用复制算法进行垃圾回收，所以新生代10M的内存中，8M是Eden，1M是Survivor，另外的1M是未使用配合复制算法的内存块，也是Survivor。

```
 1 public class ReflectTest {
 2 
 3     private static final int _1MB = 1024*1024;
 4     
 5     public static void testAllocation(){
 6         byte[] allocation1 , allocation2 , allocation3 , allocation4;
 7         allocation1 = new byte[2 * _1MB];
 8         allocation2 = new byte[2 * _1MB];
 9         allocation3 = new byte[2 * _1MB];
10         allocation4 = new byte[6 * _1MB];
11     }
12     
13     public static void main(String[] args) {
14         ReflectTest.testAllocation();
15     }
16     
17 }
```
 输出如下

```
Heap
 PSYoungGen      total 9216K, used 6651K [0x000000000b520000, 0x000000000bf20000, 0x000000000bf20000)
  eden space 8192K, 81% used [0x000000000b520000,0x000000000bb9ef28,0x000000000bd20000)
  from space 1024K, 0% used [0x000000000be20000,0x000000000be20000,0x000000000bf20000)
  to   space 1024K, 0% used [0x000000000bd20000,0x000000000bd20000,0x000000000be20000)
 PSOldGen        total 10240K, used 6144K [0x000000000ab20000, 0x000000000b520000, 0x000000000b520000)
  object space 10240K, 60% used [0x000000000ab20000,0x000000000b120018,0x000000000b520000)
 PSPermGen       total 21248K, used 2973K [0x0000000005720000, 0x0000000006be0000, 0x000000000ab20000)
  object space 21248K, 13% used [0x0000000005720000,0x0000000005a07498,0x0000000006be0000)
```
 可以看到eden占用了81%，说明allocation1 , allocation2 , allocation3 都是分配在新生代Eden上。

2、大对象直接分配在老年代上

　　大对象是指需要大量连续内存空间去存放的对象，类似于那种很长的字符串和数组。大对象对于虚拟机的内存分布来讲并不是好事，当遇到很多存活仅一轮的大对象jvm更加难处理，写代码的时候应该避免这样的问题。虚拟机中提供了-XX:PretenureSizeThreshold参数，另大于这个值的对象直接分配到老年代，这样做的目的是为了避免在Eden区和Survivor区之间发生大量的内存copy，在之前讲过的垃圾回收算法复制算法有提到过，就不多说了。

```
public class ReflectTestBig {

    private static final int _1MB = 1024*1024;
    
    public static void testAllocation(){
        byte[] allocation2 , allocation3 , allocation4;
        allocation2 = new byte[2 * _1MB];
        allocation3 = new byte[2 * _1MB];
        allocation4 = new byte[6 * _1MB];
    }
    
    public static void main(String[] args) {
        ReflectTestBig.testAllocation();
    }
    
}
```
 输出如下

```
Heap
 PSYoungGen      total 8960K, used 4597K [0x000000000b510000, 0x000000000bf10000, 0x000000000bf10000)
  eden space 7680K, 59% used [0x000000000b510000,0x000000000b98d458,0x000000000bc90000)
  from space 1280K, 0% used [0x000000000bdd0000,0x000000000bdd0000,0x000000000bf10000)
  to   space 1280K, 0% used [0x000000000bc90000,0x000000000bc90000,0x000000000bdd0000)
 PSOldGen        total 10240K, used 6144K [0x000000000ab10000, 0x000000000b510000, 0x000000000b510000)
  object space 10240K, 60% used [0x000000000ab10000,0x000000000b110018,0x000000000b510000)
 PSPermGen       total 21248K, used 2973K [0x0000000005710000, 0x0000000006bd0000, 0x000000000ab10000)
  object space 21248K, 13% used [0x0000000005710000,0x00000000059f7460,0x0000000006bd0000)
```
 可以看到allocation4已经超过了设置的-XX:PretenureSizeThreshold=3145728，随意allocation4直接被分配到了老年代，老年代占用率为60%。注意这里设置-XX:PretenureSizeThreshold=3145728不能写成-XX:PretenureSizeThreshold=3m，否则jvm将无法识别。

3、长期存活的对象将进入老年代

　　虚拟机既然采用了分带收集的思想来管理内存，那内存回收就必须识别哪些对象应该放在新生代，哪些对象应该放在老年代。为了打到目的，jvm给每个对象定义了一个年龄计数器（Age）。如果对象在Eden出生并且能过第一次Minor GC后仍然存活，并且可以在Survivor存放的话，将被移动到Survivor中，并将对象的年龄设为1。对象每躲过一次Minor GC，年龄就会加1，当他的年龄超过一年的阈值的时候，该对象就会晋升到老年代。这个阈值jvm默认是15，可以通过-XX:MaxTenuringThreshold来设置。

```
public class JavaTest {  
  
    static int m = 1024 * 1024;  
  
    public static void main(String[] args) {  
        byte[] a1 = new byte[1 * m / 4];  

　　　　 byte[] a2 = new byte[7 * m];  

　　　　 byte[] a3 = new byte[3 * m]; //GC  
    }  
}
```
 输出如下

```
[GC [DefNew: 7767K->403K(9216K), 0.0062209 secs] 7767K->7571K(19456K), 0.0062482 secs]   
[Times: user=0.00 sys=0.00, real=0.01 secs]   
a3 ok  
Heap  
 def new generation   total 9216K, used 3639K [0x331d0000, 0x33bd0000, 0x33bd0000)  
  eden space 8192K,  39% used [0x331d0000, 0x334f9040, 0x339d0000)  
  from space 1024K,  39% used [0x33ad0000, 0x33b34de8, 0x33bd0000)  
  to   space 1024K,   0% used [0x339d0000, 0x339d0000, 0x33ad0000)  
 tenured generation   total 10240K, used 7168K [0x33bd0000, 0x345d0000, 0x345d0000)  
   the space 10240K,  70% used [0x33bd0000, 0x342d0010, 0x342d0200, 0x345d0000)  
 compacting perm gen  total 12288K, used 381K [0x345d0000, 0x351d0000, 0x385d0000)  
   the space 12288K,   3% used [0x345d0000, 0x3462f548, 0x3462f600, 0x351d0000)  
    ro space 10240K,  55% used [0x385d0000, 0x38b51140, 0x38b51200, 0x38fd0000)  
    rw space 12288K,  55% used [0x38fd0000, 0x396744c8, 0x39674600, 0x39bd0000) 
```
 可以看到a2已经存活了一次，年龄为1，满足所设置的-XX:MaxTenuringThreshold=1，所以a2进入了老年代，而a3则进入了新生代。

4、动态对象年龄判定

　　为了能更好的适应不同程序的内存状态，虚拟机并不总是要求对象的年龄必须达到-XX:MaxTenuringThreshold所设置的值才能晋升到老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年区，无须达到-XX:MaxTenuringThreshold中的设置值。

5、空间分配担保

　　在发生Minor GC的时候，虚拟机会检测每次晋升到老年代的平均大小是否大于老年代的剩余空间，如果大于，则直接进行一次FUll GC。如果小于，则查看HandlerPromotionFailyre设置是否允许担保失败，如果允许那就只进行Minor GC，如果不允许则也要改进一次FUll GC。也就是说新生代Eden存不下改对象的时候就会将该对象存放在老年代。

## 三、常用的jvm参数设置
1、-Xms： 初始堆大小， 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制。

2、Xmx： 最大堆大小，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制。

3、-Xmn： 年轻代大小(1.4or lator)， 此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。
　　整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。
　　增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8。

4、-XX:NewSize： 设置年轻代大小(for 1.3/1.4)。

5、-XX:MaxNewSize： 年轻代最大值(for 1.3/1.4)。

6、-XX:PermSize： 设置持久代(perm gen)初始值。

7、-XX:MaxPermSize： 设置持久代最大值。

8、-Xss： 每个线程的堆栈大小，JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右。

9、-XX:NewRatio： 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)，-XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5。Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。

10、-XX:SurvivorRatio： Eden区与Survivor区的大小比值，设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10。

11、-XX:LargePageSizeInBytes： 内存页的大小不可设置过大， 会影响Perm的大小。

12、-XX:+DisableExplicitGC： 关闭System.gc()

13、-XX:MaxTenuringThreshold： 垃圾最大年龄，如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代. 对于年老代比较多的应用,可以提高效率.如果将此值设置为一个较大值,则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活 时间,增加在年轻代即被回收的概率该参数只有在串行GC时才有效。

14、-XX:PretenureSizeThreshold： 对象超过多大是直接在旧生代分配，单位字节 新生代采用Parallel Scavenge GC时无效另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象。

15、-XX:TLABWasteTargetPercent： TLAB占eden区的百分比。

## 四、补充
Minor GC和FUll GC的区别：

　　新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为java对象大对数都是逃不过第一轮的GC，所以Minor GC使用很频繁，一般回收速度也比较快。

　　老年代GC（FULL GC/Major GC） ：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对，在ParallelScavenge收集器的收集策略中就有直接进行Major GC的选择过程 ）。Major GC的速度一般会比Minor GC慢10倍以上。 