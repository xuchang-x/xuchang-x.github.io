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

# 第三章 垃圾收集器于内存分配策略
- -Xnoclassgc:不心境类回收
- -verbose:class
- -XX:+TraceClassLoading 追踪类加载信息 product版虚拟机可用
- -XX:+TraceClassUnLoading 追踪类卸载信息 FastDebug版虚拟机支持
