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
-