---
title: 深入理解Java虚拟机 阅读笔记
date: 2020-07-17 14:45:00
tags:
    - java
    - 虚拟机
    - 阅读笔记
description: description
---
- -Xms20m:最小堆内存20M
- -Xmx20m:最大堆内存20M
- -XX:+HeapDumpOnOutOfMemoryError:OOM时候Dump出当前内存堆转储快找一边时候进行分析

- -Xoss:设置本地方法栈大小(Hotspot虚拟机中不区分虚拟机栈和本地方法栈)
- -Xss
- -XX:PermSize:永久区大小
- -XX:MaxPermSize:永久区最大大小 限制方法区从而间接限制其中的常量池
- -XX:MaxDirectMemorySize:最大直接内存，若不指定于Java堆内存一样(-Xmx)

- -XX:+EliminateLocks可以开启同步消除,进行测试执行的效率
- -XX:-DoEscapeAnalysis关闭逃逸分析
- -XX:+EliminateAllocations可以开启标量替换
- -XX:+PrintEliminateAllocations查看标量替换情况（Server VM 非Product版本支持）
- -XX:+/-UseTLAB设定是否开启本地线程分配缓冲（TLAB）

# 第三章 垃圾收集器于内存分配策略
- -Xnoclassgc:不进行类回收
- -verbose:class
- -XX:+TraceClassLoading 追踪类加载信息 product版虚拟机可用
- -XX:+TraceClassUnLoading 追踪类卸载信息 FastDebug版虚拟机支持
- -XX:SurvivorRatio：Eden与Survivor区的比例
- -XX:PretenureSizeThreshold：直接晋升到老年代的对象大小
- -XX:PretenureSizeThreshold：晋升老年代对象年龄
- -XX:HandlePromotionFailure：是否允许分配担保失败，即老年代剩余空间不足以应付新生代的整个eden区和survivor区都存活的极端情况
- -XX:ParallelGCThreads 限制垃圾收集线程数
- -XX:+UseConcMarkSweepGC
- -XX:+UseParNewGC 强制指定ParNew收集器
- -XX:MaxGCPauseMillis：最大垃圾收集停顿时间，尽可能保证，仅在Parallel Scavenge生效
- -XX:GCTimeRatio：Parallel Scavenge设置吞吐量
- -XX:+UseAdaptiveSizePolicy：动态调整Java堆中各个区域大小以及进入老年代的年龄。 Parallel Scavenge 这个参数打开之后，就不需要手工指定新生代的大小(-Xmn)、 Eden与Survivor区的比例(-XX:SurvivorRatio)、晋升老年代对象年龄(-XX: PretenureSizeThreshold)等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信 息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量，这种调节方式称为GC 自适应的调节策略(GC Ergonomics)
- -XX:CMSInitiatingOccupancyFraction：CMS中触发full gc的老年代比例
- -XX:+UseCMSCompactAtFullCollection：默认开启，开启在CMS收集器顶不住要进行FullGC时开启内存碎片的合并 整理过程
- -XX:CMSFullGCsBeforeCompaction：执行多少次不压缩的Full GC后，跟着来一次带压缩的(默认值为0，表示每次进入Full GC时都进行碎片整理)。
- -XX:+PrintGCDetails等参数来查看GC日志
- -XX:MaxGCPauseMillis=50 GC最大停顿时间
- -UseSerialGC 使用Serial + SerialOld
- -UseParNewGC 使用ParNew + SerialOld
- -UseConcMarkSweepGC 使用ParNew+CMS+SerialOld（SerialOld兜底）
- -UseParallelGC 使用Parallel Scavenge+ Serial Old
- -UserParallelOldGC 使用Parallel Scavenge+ Parallel Old
