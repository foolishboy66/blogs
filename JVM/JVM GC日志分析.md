# JVM GC日志分析

[TOC]

GC日志是处理JVM相关问题的重要根据，所以看懂并分析GC日志是一项必须可少的技能。

## GC相关的JVM参数设置

```json
-XX:PrintGC				输出GC日志
-XX:PrintGCDetails		输出GC的详细日志
-XX:PrintGCTimeStamps	输出GC的时间戳
-XX:PrintGCDateStamps	输出GC的详细时间信息(格式为2021-04-26T23:00:18.288+0800)
-XX:PrintHeapAtGC		在GC发生的前后打印出堆的信息
-Xloggc:../logs/gc.log	生成GC日志文件的路径
```



## GC日志的含义

- 使用的命令行参数如下

  ```json
  -XX:+UseParallelGC 
  -XX:+PrintGCDetails
  -XX:+PrintGCDateStamps
  -XX:+PrintHeapAtGC
  -Xloggc:gc.log
  ```

- GC日志样例

  ```
  Java HotSpot(TM) 64-Bit Server VM (25.151-b12) for windows-amd64 JRE (1.8.0_151-b12), built on Sep  5 2017 19:33:46 by "java_re" with MS VC++ 10.0 (VS2010)
  Memory: 4k page, physical 16711680k(10587452k free), swap 17760256k(9790820k free)
  CommandLine flags: -XX:InitialHeapSize=267386880 -XX:MaxHeapSize=4278190080 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
  {Heap before GC invocations=1 (full 0):
   PSYoungGen      total 76288K, used 3932K [0x000000076b000000, 0x0000000770500000, 0x00000007c0000000)
    eden space 65536K, 6% used [0x000000076b000000,0x000000076b3d70e0,0x000000076f000000)
    from space 10752K, 0% used [0x000000076fa80000,0x000000076fa80000,0x0000000770500000)
    to   space 10752K, 0% used [0x000000076f000000,0x000000076f000000,0x000000076fa80000)
   ParOldGen       total 2785280K, used 2097152K [0x00000006c1000000, 0x000000076b000000, 0x000000076b000000)
    object space 2785280K, 75% used [0x00000006c1000000,0x0000000741000020,0x000000076b000000)
   Metaspace       used 2760K, capacity 4486K, committed 4864K, reserved 1056768K
    class space    used 302K, capacity 386K, committed 512K, reserved 1048576K
  2021-04-27T00:29:10.817+0800: 1.389: [GC (Allocation Failure) [PSYoungGen: 3932K->752K(76288K)] 2101084K->2097912K(2861568K), 0.0261894 secs] [Times: user=0.13 sys=0.00, real=0.03 secs] 
  Heap after GC invocations=1 (full 0):
   PSYoungGen      total 76288K, used 752K [0x000000076b000000, 0x0000000774500000, 0x00000007c0000000)
    eden space 65536K, 0% used [0x000000076b000000,0x000000076b000000,0x000000076f000000)
    from space 10752K, 6% used [0x000000076f000000,0x000000076f0bc020,0x000000076fa80000)
    to   space 10752K, 0% used [0x0000000773a80000,0x0000000773a80000,0x0000000774500000)
   ParOldGen       total 2785280K, used 2097160K [0x00000006c1000000, 0x000000076b000000, 0x000000076b000000)
    object space 2785280K, 75% used [0x00000006c1000000,0x0000000741002020,0x000000076b000000)
   Metaspace       used 2760K, capacity 4486K, committed 4864K, reserved 1056768K
    class space    used 302K, capacity 386K, committed 512K, reserved 1048576K
  }
  ...
  {Heap before GC invocations=3 (full 1):
   PSYoungGen      total 141824K, used 712K [0x000000076b000000, 0x0000000774500000, 0x00000007c0000000)
    eden space 131072K, 0% used [0x000000076b000000,0x000000076b000000,0x0000000773000000)
    from space 10752K, 6% used [0x0000000773a80000,0x0000000773b32020,0x0000000774500000)
    to   space 10752K, 0% used [0x0000000773000000,0x0000000773000000,0x0000000773a80000)
   ParOldGen       total 2785280K, used 2097160K [0x00000006c1000000, 0x000000076b000000, 0x000000076b000000)
    object space 2785280K, 75% used [0x00000006c1000000,0x0000000741002020,0x000000076b000000)
   Metaspace       used 2760K, capacity 4486K, committed 4864K, reserved 1056768K
    class space    used 302K, capacity 386K, committed 512K, reserved 1048576K
  2021-04-27T00:29:10.867+0800: 1.438: [Full GC (Allocation Failure) [PSYoungGen: 712K->0K(141824K)] [ParOldGen: 2097160K->561K(201728K)] 2097872K->561K(343552K), [Metaspace: 2760K->2760K(1056768K)], 0.1741962 secs] [Times: user=0.14 sys=0.14, real=0.17 secs] 
  Heap after GC invocations=3 (full 1):
   PSYoungGen      total 141824K, used 0K [0x000000076b000000, 0x0000000774500000, 0x00000007c0000000)
    eden space 131072K, 0% used [0x000000076b000000,0x000000076b000000,0x0000000773000000)
    from space 10752K, 0% used [0x0000000773a80000,0x0000000773a80000,0x0000000774500000)
    to   space 10752K, 0% used [0x0000000773000000,0x0000000773000000,0x0000000773a80000)
   ParOldGen       total 201728K, used 561K [0x00000006c1000000, 0x00000006cd500000, 0x000000076b000000)
    object space 201728K, 0% used [0x00000006c1000000,0x00000006c108c620,0x00000006cd500000)
   Metaspace       used 2760K, capacity 4486K, committed 4864K, reserved 1056768K
    class space    used 302K, capacity 386K, committed 512K, reserved 1048576K
  }
  ```

  GC前后打印的堆信息分别是GC前后堆内存的快照信息，其中，```Heap before GC invocations=1 (full 0):```部分是GC前的堆信息；```Heap after GC invocations=1 (full 0):```是GC后的堆信息；两者之间的是本次GC的详细信息。

- Minor GC日志分析

  ```json
  2021-04-27T00:29:10.817+0800: 1.389: [GC (Allocation Failure) [PSYoungGen: 3932K->752K(76288K)] 2101084K->2097912K(2861568K), 0.0261894 secs] [Times: user=0.13 sys=0.00, real=0.03 secs] 
  ```

  1. "2021-04-27T00:29:10.817+0800"是GC发生的详细时间节点，"1.389"是从JVM启动以来经过的秒数。
  2. "PSYoungGen"是GC类型，用来区分是新生代GC还是老年代GC，PSYoungGen代表是新生代GC。
  3. **aK->bK(cK)**的形式表示的含义是GC前该区域已使用的空间容量为a，GC后该区域的使用容量为b，该区域的总容量为c。相应地，"3932K"是GC前该区域已使用的空间容量，752K是GC后该区域的使用容量，76288K是该区域的总容量；2101084K是GC前Java堆已使用的空间容量，2097912K是GC后Java堆已使用的空间容量，2861568K是Java堆的总容量。
  4. Times部分代表该内存本次GC相关的耗时，单位是秒。user代表用户态消耗CPU的时间；sys代表内核消耗的CPU时间；real代表GC操作从开始到结束所经过的墙钟时间。

- Full GC日志分析

  ```json
  2021-04-27T00:29:10.867+0800: 1.438: [Full GC (Allocation Failure) [PSYoungGen: 712K->0K(141824K)] [ParOldGen: 2097160K->561K(201728K)] 2097872K->561K(343552K), [Metaspace: 2760K->2760K(1056768K)], 0.1741962 secs] [Times: user=0.14 sys=0.14, real=0.17 secs] 
  ```

  1. PSYoungGen代表是新生代GC；ParOldGen代表是老年代GC

  2. Metaspace代表是元空间的GC，0.1741962 secs代表元空间GC操作的耗时

     

## GC时间分析

墙钟时间包括各种非运算的等待耗时，例如等待磁盘IO、等待线程阻塞等，而CPU时间不包含这些，但是当系统有多CPU或者多核的时候 ，多线程操作会叠加这些CPU时间，所以有时候会出现user或sys的时间超过real时间的现象。

1. 通常情况下，垃圾回收的过程是并发执行的，一般都会是user+sys>real

2. 当使用了串行垃圾回收器时，正常情况下应该是user+sys=real

3. 当出现real远大于user+sys时，可能是由于频繁的IO（网络IO或磁盘IO）操作或者缺乏CPU资源引起的

- IO密集型

  因为记录GC日志也是磁盘读写，当同一时间其它IO操作比较多，会导致GC事件延迟，从而影响最后的real时间。程序本身的IO或者同一机器上其它进程的频繁IO都会影响最终的real值，当频繁的IO操作确实存在时，可以采用以下两种方式改进：一是优化程序本身的IO逻辑；二是消除其它进程IO对当前程序的影响。

- CPU密集型

  当程序本身是CPU密集型应用或者同意机器上其它进程占用了大量的CPU资源，有可能会出现当前程序分配不到CPU资源的情况，会导致real值远大于user+sys。可以采用以下方式改进：一是优化程序内部的线程使用，确保无冗余线程；二是增加虚拟机或者容器本身的CPU配置，提升机器总的计算能力。



## 参考文献

深入理解Java虚拟机