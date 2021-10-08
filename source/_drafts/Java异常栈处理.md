---
title: Java异常栈处理
date: 2021-02-01 17:39:00
tags:
    - Java
    - 异常
description: 
---
- [Java中异常的原理以及处理](https://www.cnblogs.com/yurenjun/p/13382487.html)

# 一、什么是异常
异常是程序中的一些错误，但并不是所有的错误都是异常，并且错误有时候是可以避免的。

# 二、异常的分类
## 三种类型的异常（Throwable）：
- 编译时异常：最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。

    编译器要求必须处置的异常。即程序在运行时由于外界因素造成的一般性异常。编译器要求java程序必须捕获或声明所有编译时异常。
    
- 运行时异常：运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。

    编译器不要求强制处置的异常。一般是指编程时的逻辑错误，是程序员应该积极避免其出现的异常。java.lang.RuntimeException类及它的子类都是运行时异常。

- 错误：错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如，当栈溢出时，一个错误就发生了，它们在编译也检查不到的。

    Error类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢等。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这样的错误，建议让程序终止。
    
# 三、Java 内置异常类
Java 语言定义了一些异常类在 java.lang 标准包中。

标准运行时异常类的子类是最常见的异常类。由于 java.lang 包是默认加载到所有的 Java 程序的，所以大部分从运行时异常类继承而来的异常都可以直接使用。

Java 根据各个类库也定义了一些其他的异常，下面的表中列出了 Java 的运行时异常类。

|异常|描述|
|:----|:----|
|ArithmeticException|当出现异常的运算条件时，抛出此异常。例如，一个整数"除以零"时，抛出此类的一个实例。|
|ArrayIndexOutOfBoundsException|用非法索引访问数组时抛出的异常。如果索引为负或大于等于数组大小，则该索引为非法索引。|
|ArrayStoreException|试图将错误类型的对象存储到一个对象数组时抛出的异常。|
|ClassCastException|当试图将对象强制转换为不是实例的子类时，抛出该异常。|
|IllegalArgumentException|抛出的异常表明向方法传递了一个不合法或不正确的参数。|
|IllegalMonitorStateException|抛出的异常表明某一线程已经试图等待对象的监视器，或者试图通知其他正在等待对象的监视器而本身没有指定监视器的线程。|
|IllegalStateException|在非法或不适当的时间调用方法时产生的信号。换句话说，即 Java 环境或 Java 应用程序没有处于请求操作所要求的适当状态下。|
|IllegalThreadStateException|线程没有处于请求操作所要求的适当状态时抛出的异常。|
|IndexOutOfBoundsException|指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。|
|NegativeArraySizeException|如果应用程序试图创建大小为负的数组，则抛出该异常。|
|NullPointerException|当应用程序试图在需要对象的地方使用 `null` 时，抛出该异常|
|NumberFormatException|当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。|
|SecurityException|由安全管理器抛出的异常，指示存在安全侵犯。|
|StringIndexOutOfBoundsException|此异常由 `String` 方法抛出，指示索引或者为负，或者超出字符串的大小。|
|UnsupportedOperationException|当不支持请求的操作时，抛出该异常。|

下面的表中列出了 Java 定义在 java.lang 包中的编译时异常类。

|异常|描述|
|:----|:----|
|ClassNotFoundException|应用程序试图加载类时，找不到相应的类，抛出该异常。|
|CloneNotSupportedException|当调用 `Object` 类中的 `clone` 方法克隆对象，但该对象的类无法实现 `Cloneable` 接口时，抛出该异常。|
|IllegalAccessException|拒绝访问一个类的时候，抛出该异常。|
|InstantiationException|当试图使用 `Class` 类中的 `newInstance` 方法创建一个类的实例，而指定的类对象因为是一个接口或是一个抽象类而无法实例化时，抛出该异常。|
|InterruptedException|一个线程被另一个线程中断，抛出该异常。|
|NoSuchFieldException|请求的变量不存在|
|NoSuchMethodException|请求的方法不存在|

# 四、怎么处理异常
Java提供的是异常处理的抓抛模型。
1."抛"：当我们执行代码时，一旦出现异常，就会在异常代码处生成一个对应的异常类型的对象，并将此对象抛出。该异常对象将被提交给Java运行时系统，这个过程称为抛出(throw)异常。

异常对象的生成:

由虚拟机自动生成：程序运行过程中，虚拟机检测到程序发生了问题，如果在当前代码中没有找到相应的处理程序，就会在后台自动创建一个对应异常类的实例对象并抛出——自动抛出

2."抓":抓住上一步跑出来的异常类对象，如何抓？即为异常的处理方式。

处理方式一：try{}catch(){}finally模型
```java
try{
  ......  //可能产生异常的代码
}
catch( ExceptionName1 e ){
  ......  //当产生ExceptionName1型异常时的处置措施
}
catch( ExceptionName2 e ){
......   //当产生ExceptionName2型异常时的处置措施
} 
[ finally{
......   //无论是否发生异常，都无条件执行的语句
  }  ]
```
处理方式二：声明抛出异常是Java中处理异常的第二种方式
```java
public class A {
 	public void methodA() throws IOException {
 	      ……
 	}  }
    public class B1 extends A {
 	public void methodA() throws FileNotFoundException {
 	      ……
 	}  }
    public class B2 extends A {
 	public void methodA() throws Exception {   //报错
 	        ……
  }  }
```