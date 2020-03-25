# 第4章 虚拟机性能监控、故障处理工具

## 4.1 概述
- 给一个系统定位问题的时候，知识、经验是关键基础，数据是依据，工具是运用知识处理数据的手段。这里说的数据包括但不限于异常堆栈、虚拟机运行日志、
垃圾收集器日志、线程快照（threaddump/javacore文件）、堆转储快照（heapdump/hrof文件）等。恰当地使用虚拟机故障处理、分析的工具可以提升我们分析数据、
定位并解决问题的效率。

## 4.2 基础故障处理工具
- 在JDK的bin目录下有各种小工具，这些主要是用于监视虚拟机运行状态和进行故障处理的工具，根据软件可用性和授权不同，可以把它们划分为三类：
  - 商业授权工具：主要是JMC（Java Mission Control）及它要使用到的JFR（Java Flight Recorder）。
  - 正式支持工具：这一类是属于被长期支持的工具。
  - 实验性工具：这一类工具在它们使用说明中被声明为"没有技术支持，并且是实验性质的"产品，日后可能会转正，也可能会在某个JDK版本中无声无息地消失。
  但事实上它们通常都非常稳定而且功能强大，也能在处理应用程序性能问题、定位故障时发挥很大作用。
  
### 4.2.1 jps：虚拟机进程状况工具
- jps:可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名称以及这些进程的本地虚拟机唯一ID（LVMID，
Local Virtual Machine Identifier）。
- jps命令格式
```text
jps [ options ] [ hostid ]
```
jps执行样例
jps -l
```text
jps -l                                                                                                                                                                                                                                
1177 org.jetbrains.idea.maven.server.RemoteMavenServer36
1481 jdk.jcmd/sun.tools.jps.Jps
1147 
```
- jps还可以通过RMI协议查询开启了RMI服务的远程虚拟机进程状态，参数hostid为RMI注册表中注册的主机名。


### 4.2.2 jstat：虚拟机统计信息监视工具
- jstat(JVM Statistics Monitoring Tool)是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类加载、内存、
垃圾收集、即时编译等运行时数据，在没有GUI图形界面、只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的常用工具。
- jstat命令格式为：
```text
jstat [ option vmid [interval[s|ms] [count]] ]
```
- 对于命令格式中的VMID与LVMID需要特别说明一下：如果是本地虚拟机进程，VMID与LVMID是一致的；如果是远程虚拟机进程，那VMID的格式应当是：
```text
[protocol:][//]lvmid[@hostname[:port]/servername]
```
- 参数interval和count代表查询间隔和次数，如果省略这2个参数，说明只查询一次。假设需要每250毫秒查询一次进程2764垃圾收集情况，一共查询20次，
那命令应当是：
```text
jstat -gc 2764 250 20
```
- 选项option代表用户希望查询的虚拟机信息，主要分为三类：类加载、垃圾收集、运行期编译状况。
- jstat执行样例
```text
jstat -gcutil 7304
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT   
  0.00  44.06  39.52  70.35  93.62  88.93    103    0.644    12    0.322     -        -    0.966
```
- 查询结果表明：该进程的新生代Eden（E，表示Eden）使用了39.52%的空间，2个Survivor区（S0、S1，表示Survivor0、Survivor1）分别使用了0和44.06%的空间。
老年代（O，表示 Old）使用了70.35%的空间。程序运行以来共发生Minor GC（YGC，表示Yong GC）16次，总耗时0.644秒；发生Full GC（FGC，表示Full GC）12次，
总耗时（FGCT，表示Full GC Time）为0.322秒；所有GC总耗时（GCT，表示 GC Time）为0.966秒。


### 4.2.3 jinfo：Java配置信息工具 
- jinfo（Configuration Info for Java）的作用是实时查看和调整虚拟机各项参数。jinfo的-flag选项可以查看虚拟机启动时未被显式指定的参数的系统默认值。
jinfo还可以使用-sysprops选项把虚拟机进程的System.getProperties()的内容打印出来。
- jinfo命令格式：
```text
jinfo [ option ] pid
```
- 执行样例：查询CMSInitiatingOccupancyFraction参数值
```text
jinfo -flag CMSInitiatingOccupancyFraction 7304
-XX:CMSInitiatingOccupancyFraction=-1
```

### 4.2.4 jmap：Java内存映像工具
- jmap（Memory Map for Java）命令用于生成堆转储快照（一般称为heapdump或dump文件）。jmap还可以查询finalize执行队列、Java堆和方法区的详细信息，
如空间使用率、当前用的是哪种收集器等。
- jmap命令格式：
```text
jmap [ option ] vmid
```
- 执行样例：
```text
jmap -dump:format=b,file=idea.bin 7304                                                                                                                                                                       [2d5h7m] ✹ ✭
Heap dump file created
```


### 4.2.5 jhat：虚拟机堆转储快照分析工具
- JDK提供jhat（JVM Heap Analysis Tool）命令与jmap搭配使用，来分析jmap生成的堆转储快照。jhat内置了一个微型的HTTP/Web服务器，
生成的堆转储快照的分析结果后，可以在浏览器查看。
- 使用jhat分析dump文件
```text
jhat idea.bin
Reading from idea.bin...
Dump file created Mon Mar 23 23:39:03 CST 2020
Snapshot read, resolving...
Resolving 2817601 objects...
Chasing references, expect 563 dots......
Eliminating duplicate references.......
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```
- 屏幕显示"Server is ready."的提示后，用户在浏览器中输出http://localhost:7000/可以看到分析结果。
