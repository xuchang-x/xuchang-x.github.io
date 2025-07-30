---
title: JAVA业务开发常见错误
date: 2021-11-09 17:48:20
tags:
    - JAVA
    - 学习笔记
    - 极客时间
---
- 线程池类型 不同参数意义
- 多个线程池类型优劣势

- ConcurrentHashMap特性&代码
- 方法级的synchronized和字段上的synchronized差异，JVM字节码上有什么差异
- 代码块级别的synchronized和方法上标记synchronized关键字，在实现上有什么区别

|  分类    | 被锁的对象  | 伪代码 |
|  :----:  | :----:  | :----:  |
| 实例方法  | 类的实例对象 | public synchronized void method(){...}|
| 静态方法  | 类对象       | public static synchronized void method(){...} |
| 实例对象  | 类的实例对象 | synchronized(this){...} |
| class对象  | 类对象 | synchronized(SynchronizedDemo.class){...} |
| 任意对象Object  | 配置的实例对象Object | synchronized(lock){...} |

