[TOC]

------

### 1.1 参数分类及说明



在使用 JDK 命令行工具时，会用到各种各样的参数，我们以 `java` 命令的[官方参考文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)为参考，看一看 JVM 命令行的参数如何分类



1. Standard Options 标准参数

   JVM 的所有实现都保证标准选项得到支持。 它们用于常见操作，例如检查JRE的版本，设置类路径，启用详细输出等

2. Non-Standard Options 非标准参数

   非标准选项是特定于 Java HotSpot 虚拟机的通用选项，因此不能保证所有JVM实现都支持它们，它们可能会发生变化。这些选项以 `-X`开头

3. Advanced Options 高级参数

   不建议将高级选项用于临时使用。 这些是用于调整Java HotSpot虚拟机操作的特定区域的开发人员选项，这些区域通常具有特定的系统要求，并且可能需要对系统配置参数的特权访问。 它们也不能保证得到所有JVM实现的支持，并且可能会发生变化。 高级选项以 `-XX` 开头。

   - Advanced Runtime Options 高级运行时参数
   - Advanced JIT Compiler Options 高级即时编译参数
   - Advanced Serviceability Options 高级可维护参数
   - Advanced Garbage Collection Options 高级垃圾回收参数



------

### 1.2. 挑个最简单的命令试一试

相信每一个 java 使用者最早使用的 java 命令就是 `java -version`，让我们看一下这个命令的官方说明：显示版本信息，然后退出。这个选项相当于 `-showversion` 选项，但后者不会在显示版本信息后指示JVM退出。可以看出，该命令实际上是启动了 JVM 虚拟机，输出版本信息后推出了虚拟机，那么，配合 `-X` 参数我们可以验证，默认的编译模式是 mixed mode。

> - -Xint 解释执行
> - -Xcomp 第一次使用就编译成本地代码
> - -Xmixed 混合模式，JVM 自己决定是否编译成本地代码

```shell
~ » java -Xint -version                                                                                                lishaofei@bogon
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, interpreted mode)
------------------------------------------------------------
~ » java -Xcomp -version                                                                                               lishaofei@bogon
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, compiled mode)
------------------------------------------------------------
~ » java -Xmixed -version                                                                                              lishaofei@bogon
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
------------------------------------------------------------
~ » java -version                                                                                                      lishaofei@bogon
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```



------

### 1.3  高级参数如何使用？

布尔型的参数使用 +/- 表示启用或禁用，如：`-XX:+UseConcMarkSweepGC` 表示启用CMS垃圾回收器，`-XX:+UseG1GC` 表示启用G1垃圾回收器



非布尔型的参数使用 `-XX:<name>=<value>` ，表示name属性的值是value，如：`-XX:MaxGCPauseMillis=500` 表示GC的最大停顿时间为500ms



我们最常用的 `-Xms` 等价于 `-XX:InitialHeapSize`，`-Xmx` 等价于 `-XX:MaxHeapSize`



我们可以使用 `jinfo` 命令查看运行中的 java 程序的相关参数，示例如下



```shell
/Library/Preferences » jinfo -flag MaxHeapSize 674                                                                     lishaofei@bogon
-XX:MaxHeapSize=734003200
------------------------------------------------------------
/Library/Preferences » jinfo -flag ThreadStackSize 674                                                                 lishaofei@bogon
-XX:ThreadStackSize=1024
```



---

### 1.4 通过设置高级参数打印 JVM 参数信息

- `-XX:+PrintFlagsInitial` 查看初始值
- `-XX:+PrintFlagsFinal` 查看修改值
- `-XX:+UnlockExperimentalVMOptions` 解锁实验参数
- `-XX:+UnlockDiagnosticVMOptions` 解锁诊断参数
- `-XX:+PrintCommandLineFlags` 打印命令行参数

```
java -XX:+PrintFlagsFinal -version > flags.txt
```

展示结果中，=表示默认值，:=表示被用户或JVM修改后的值



---

### 1.5 `jps`的使用



`jps` ([官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html#CHDCGECD)) ，类似于 linux 中的 ps

```shell
/Library/Preferences » jps -l                                                                                          lishaofei@bogon
674 org.jetbrains.jps.cmdline.Launcher
5187 sun.tools.jps.Jps
361
668 org.jetbrains.idea.maven.server.RemoteMavenServer
```



---

### 1.6 `jinfo` 查看运行时参数

`jinfo` 查看最大内存和垃圾回收器信息

```shell
/Library/Preferences » jinfo -flag MaxHeapSize 668                                                                     lishaofei@bogon
-XX:MaxHeapSize=805306368
------------------------------------------------------------
/Library/Preferences » jinfo -flag UseConcMarkSweepGC 668                                                              lishaofei@bogon
-XX:-UseConcMarkSweepGC
------------------------------------------------------------
/Library/Preferences » jinfo -flag UseG1GC 668                                                                         lishaofei@bogon
-XX:-UseG1GC
------------------------------------------------------------
/Library/Preferences » jinfo -flag UseParallelGC 668                                                                   lishaofei@bogon
-XX:+UseParallelGC
```



---

### 1.7 `jstat` 查看JVM统计信息

`jstat` 查看类装载信息，1000 3 表示 1000 毫秒刷新一次，共 3 次

```shell
/Library/Preferences » jstat -class 668 1000 3                                                                         lishaofei@bogon
Loaded  Bytes  Unloaded  Bytes     Time
  3521  6170.2       30    42.8       1.12
  3521  6170.2       30    42.8       1.12
  3521  6170.2       30    42.8       1.12
```

 

使用 `-gc` 查看垃圾回收信息

```shell
/Library/Preferences » jstat -gc 668 1000 3                                                                            lishaofei@bogon
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
10752.0 10752.0  0.0    0.0   65536.0   6020.4   175104.0    4644.4   18432.0 17914.1 2304.0 2134.3      3    0.021   2      0.050    0.071
10752.0 10752.0  0.0    0.0   65536.0   6020.4   175104.0    4644.4   18432.0 17914.1 2304.0 2134.3      3    0.021   2      0.050    0.071
10752.0 10752.0  0.0    0.0   65536.0   6020.4   175104.0    4644.4   18432.0 17914.1 2304.0 2134.3      3    0.021   2      0.050    0.071
```

- S0C、S1C、S0U、S1U：S0和S1的总量与使用量
- EC、EU：Eden区总量与使用量
- OC、OU：Old区总用量与使用量
- MC、MU：Metaspace区总量与使用量
- CCSC、CCSU：压缩类空间总量与使用量
- YGC、YGCT：YoungGC的次数与时间
- FGC、FGCT：FullGC的次数与时间
- GCT：总的GC时间

![image-20190617171027722](/Users/lishaofei/Documents/GitHub/zerolee1993/gitbook/markdown/java/draft/assets/image-20190617171027722.png)



使用 `-gcutil`查看垃圾回收概要信息

```shell
/Library/Preferences » jstat -gcutil 668 1000 3                                                                        lishaofei@bogon
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  10.25   2.65  97.19  92.63      3    0.021     2    0.050    0.071
  0.00   0.00  10.25   2.65  97.19  92.63      3    0.021     2    0.050    0.071
  0.00   0.00  10.25   2.65  97.19  92.63      3    0.021     2    0.050    0.071
```



`-gccause`，`-gcnew`，`-gcold`，参考文档有很多



使用 `-compiler` 查看 JIT 即时编译信息

```shell
/Library/Preferences » jstat -compiler 668 1000 3                                                                      lishaofei@bogon
Compiled Failed Invalid   Time   FailedType FailedMethod
    2162      0       0     2.12          0
    2162      0       0     2.12          0
    2162      0       0     2.12          0
------------------------------------------------------------
/Library/Preferences » jstat -printcompilation 668 1000 3                                                              lishaofei@bogon
Compiled  Size  Type Method
    2167     71    1 java/io/ObjectOutputStream writeTypeString
    2167     71    1 java/io/ObjectOutputStream writeTypeString
    2167     71    1 java/io/ObjectOutputStream writeTypeString
```



---

### 1.8 内存溢出的处理

导出内存映像文件的方式

- 内存溢出时自动导出，设置参数：`-XX:+HeapDumpOnOutOfMemoryError`, ` -XX:HeapDumpPath=/tmp/`

- `jmap` 命令手动导出

  ```shell
  jmap -dump:format=b,file=/tmp/heap.hprof 9060
  ```

使用 MAT 分析内存溢出的原因



---

### 1.9 死锁和死循环的处理

使用`jstack` 导出线程信息

```shell
jstack -l 9060 > jstack.txt
```

可以通过[在线分析工具](https://fastthread.io/)进行分析



[线程状态官网说明](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr034.html)

[线程状态转化参考](https://mp.weixin.qq.com/s/GsxeFM7QWuR--Kbpb7At2w)



`top -p 4361 -H` 查看进程的所有线程

`printf "%x" 15849` 打印占用 cpu 较高的线程十六进制编号

在线程日志分析工具中搜索编号定位线程



死锁问题可以直接导出日志，并在日志分析工具中分析

![image-20190618121359277](/Users/lishaofei/Documents/GitHub/zerolee1993/gitbook/markdown/java/draft/assets/image-20190618121359277.png)