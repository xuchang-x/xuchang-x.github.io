---
title: Java异常信息流转
date: 2021-05-11 20:21:00
tags:
    - Java
    - 异常栈
description: 通常我们会使用try-catch块捕获检查型异常并通过e.printStackTrace()在控制台输出异常栈信息。本文按照异常触发流程、信息填充流程、异常捕获流程、信息输出流程四个步骤，对异常产生以及输出的过程进行深入分析。
---
通常我们会使用try-catch块捕获检查型异常并通过e.printStackTrace()在控制台输出异常栈信息。本文按照异常触发流程、信息填充流程、异常捕获流程、信息输出流程四个步骤，对异常产生以及输出的过程进行深入分析。

# 核心类介绍

## Throwable

![异常分类](异常栈-异常类.png)

Throwable类是Java中一切Exception类与Error类的父类，它直接以Native方法与jvm进行交互，从jvm中获取java程序运行时的异常和错误信息。

Java程序无法对Error类型的异常进行捕获处理，Error类异常通常为jvm虚拟机本身问题，如系统崩溃、虚拟机错误、动态链接失败等。一般的异常处理是指Excepiton类型异常处理。Exception可以分为运行时异常RuntimeException和非运行时异常。

异常的另一种分类方式，分为检查型异常和非检查型异常，unchecked异常包括java.lang.RuntimeException、java.lang.Error以及它们的子类，checked exception需要在代码中进行显示处理。

![Throwable类图](异常栈-Throwable类图.png)

Throwable类中主要通过cause参数存储引发当前异常的异常，也就是当前异常的内部异常。

stackTrace参数主要用于存储当前异常触发时执行点的栈信息。

## StackTraceElement
![StackTraceElement类图](异常栈-StackTraceElement类图.png)

StackTraceElement类中主要有四个参数，用于存储运行过程中的栈信息。

declaringClass存储执行点所属类的全限定名，methodName执行点所属方法名称，fileName存储执行点所属源文件名称，lineNumber存储执行点在源文件中的行号。

# 异常触发流程
Java虚拟机引发异常是由于以下三个原因之一：
1. 执行了一条athrow指令。
2. 同步异常：
    - 将异常认定为正常情况（非检查异常），例如数组越界、JVM加载过程中发生错误
    - 资源超限，OOM、StackOverflow
3. 异步异常
    - 调用了Thread或ThreadGroup类的stop方法
    - JVM内部异常

## athrow指令
athrow指令由java代码的throw关键字产生，以如下代码为例
```java
public class SimpleTryCatch {
    public static void main(String[] args) {
        try {
            throw new RuntimeException("test");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
反编译后可以看到主函数偏移量9位置为athrow操作。
```text
public class com.example.demo.SimpleTryCatch {
  public com.example.demo.SimpleTryCatch();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/RuntimeException
       3: dup
       4: ldc           #3                  // String test
       6: invokespecial #4                  // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
       9: athrow
      10: astore_1
      11: aload_1
      12: invokevirtual #6                  // Method java/lang/Exception.printStackTrace:()V
      15: return
    Exception table:
       from    to  target type
           0    10    10   Class java/lang/Exception
}
```

## 将异常认定为正常情况（非检查异常）异常如何抛出

# 信息填充流程
![信息填充流程](异常栈-填充流程.png)

如上文所述，Throwable主要通过cause和stackTrace两个参数存储异常相关信息。

## 填充内部异常 cause参数
引发当前异常的异常可以作为cause参数，在构造方法中传入。
```java
    public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }
```
## 填充栈信息 stackTrace参数
调用native方法，填充异常发生时当前jvm stack信息。
```java
    public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }

    public synchronized Throwable fillInStackTrace() {
        if (stackTrace != null ||
            backtrace != null /* Out of protocol state */ ) {
            fillInStackTrace(0);
            stackTrace = UNASSIGNED_STACK;
        }
        return this;
    }

    private native Throwable fillInStackTrace(int dummy);
```

## native方法 fillInStackTrace
栈区信息获取核心依赖native方法fillInStackTrace，大致过程是从虚拟栈中遍历栈帧并构造StackTraceElement返回。
```
// private native Throwable fillInStackTrace(int dummy);
// (I)Ljava/lang/Throwable;
func fillInStackTrace(frame *rtda.Frame) {
    this := frame.LocalVars().GetThis()
    frame.OperandStack().PushRef(this)

    stes := createStackTraceElements(this, frame.Thread())
    this.SetExtra(stes)
}

func createStackTraceElements(tObj *heap.Object, thread *rtda.Thread) []*StackTraceElement {
    skip := distanceToObject(tObj.Class()) + 2
    frames := thread.GetFrames()[skip:]
    stes := make([]*StackTraceElement, len(frames))
    for i, frame := range frames {
        stes[i] = createStackTraceElement(frame)
    }
    return stes
}
```

# 异常捕获流程
构造如下代码：
```java
public class ExceptionTable {
    public static void main(String[] args) {
        try {
            throw new NullPointerException("npe test");
        } catch (NullPointerException e) {
            System.out.println("NPE");
            e.printStackTrace();
            throw new RuntimeException("runtime exception test");
        } catch (RuntimeException e) {
            System.out.println("runtime exception");
            e.printStackTrace();
        }
    }
}
```
反编译后得到如下文件：
```text
public class com.example.demo.ExceptionTable {
  public com.example.demo.ExceptionTable();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/NullPointerException
       3: dup
       4: ldc           #3                  // String npe test
       6: invokespecial #4                  // Method java/lang/NullPointerException."<init>":(Ljava/lang/String;)V
       9: athrow
      10: astore_1
      11: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
      14: ldc           #6                  // String NPE
      16: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      19: aload_1
      20: invokevirtual #8                  // Method java/lang/NullPointerException.printStackTrace:()V
      23: new           #9                  // class java/lang/RuntimeException
      26: dup
      27: ldc           #10                 // String runtime exception test
      29: invokespecial #11                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      32: athrow
      33: astore_1
      34: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
      37: ldc           #12                 // String runtime exception
      39: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      42: aload_1
      43: invokevirtual #13                 // Method java/lang/RuntimeException.printStackTrace:()V
      46: return
    Exception table:
       from    to  target type
           0    10    10   Class java/lang/NullPointerException
           0    10    33   Class java/lang/RuntimeException
}
```
可以看到主字节码序列中的athrow指令会抛出RuntimeException，Java虚拟机将通过在异常表中查找相关异常来跳转到实现catch子句的字节码序列。

捕获异常的每个方法都与一个异常表相关联，该异常表与该方法的字节码序列一起存储在类文件中。
对于每个try块的catch语句，异常表都有一个条目。每个条目都有四段信息：起点和终点，要跳转到的字节码序列中偏移量以及要捕获的异常类的常量池索引。

如示例中的NullPointerException捕获了偏移量0到9（含9）的数据。'to'下列出的try块的终结点值始终比捕获异常的最后偏移量大1。
在这种情况下，端点值列出为10，但是捕获到异常的最后一个pc偏移为9。此范围对应于在的try块内实现代码的字节码序列。

如果在执行方法期间引发异常，则Java虚拟机将在异常表中搜索匹配的条目。如果当前程序计数器在条目指定的范围内，并且抛出的异常类是条目指定的异常类（或者是指定的异常类的子类），则异常表条目匹配。
Java虚拟机按照条目在表中出现的顺序搜索异常表。找到第一个匹配项后，Java虚拟机将程序计数器设置为新的偏移位置，并在该位置继续执行。
如果未找到匹配项，则Java虚拟机将弹出当前堆栈帧并重新引发相同的异常。
当Java虚拟机弹出当前堆栈帧时，它有效地中止了当前方法的执行，并返回到调用此方法的方法。
但是，它没有在父级方法中正常继续执行，而是在该方法中引发了相同的异常，从而在父级方法中搜索该方法的异常表。

![信息输出流程](多次catch.jpg)
需注意的是按照上边的异常表显示，NullPointerException和RuntimeException都会捕获偏移量0到9（含9）的数据，
即使在处理NullPointerException的过程中抛出了类型相同的RuntimeException异常（偏移量32），也不会在第二个catch中进行相应处理。

## 抛出到main函数的异常
所有的高级语言都有程序初始化过程，将控制流转化为节点。初始化过程会包括
- 启动JVM
- 类加载
- 运行静态初始化程序

语言运行时(Language Runtime)会提供最外部的异常处理程序，以捕获用户代码未捕获的所有异常。
通常处理方式会打印出堆栈跟踪，有序方式关闭程序，并以错误代码退出。可以看作如下伪代码：
```java
    try {
        loadClasses();
        runInitializers();
        main(argv);
        System.exit(0);
    } catch (Throwable e) {
        e.printStackTrace();
        System.exit(-1);
    }
```

# 信息输出流程
![信息输出流程](异常栈-输出流程.png)
```java
public class CheckedException {
    public static void main(String[] args) {
        try {
            throwException();
        } catch (InnerException e) {
            e.printStackTrace();
        }
    }

    private static void throwException() throws InnerException {
        throw new InnerException("InnerException");
    }

    private static class InnerException extends Exception {
        InnerException(String message) {
            super(message);
        }
    }
}
```
以如上述一段简单的代码为例，最终输出的异常信息如下图所示
![异常信息](异常信息.jpg)
简要分析throwable打印信息相关函数代码
```java
public class Throwable implements Serializable {

    public StackTraceElement[] getStackTrace() {
        return getOurStackTrace().clone();
    }
    
    // 获取异常栈信息
    private synchronized StackTraceElement[] getOurStackTrace() {
        ……
        int depth = getStackTraceDepth();
        stackTrace = new StackTraceElement[depth];
        ……
        return stackTrace;
    }

    public void printStackTrace(PrintStream s) {
        printStackTrace(new WrappedPrintStream(s));
    }

    private void printStackTrace(PrintStreamOrWriter s) {
        ……
        synchronized (s.lock()) {
            // Print our stack trace
            // 打印当前异常类信息
            s.println(this);
            // 打印异常栈信息
            StackTraceElement[] trace = getOurStackTrace();
            for (StackTraceElement traceElement : trace)
                s.println("\tat " + traceElement);
            
            // 打印内部异常信息
            // Print cause, if any
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
        }
    }
    
    // 打印内部异常
    private void printEnclosedStackTrace(PrintStreamOrWriter s,
                                         StackTraceElement[] enclosingTrace,
                                         String caption,
                                         String prefix,
                                         Set<Throwable> dejaVu) {
        assert Thread.holdsLock(s.lock());
            ……
            // 计算内部异常与外部异常栈的层数差异
            // Compute number of frames in common between this and enclosing trace
            int framesInCommon = trace.length - 1 - m;
            
            // 打印差异层的详细信息
            // Print our stack trace
            s.println(prefix + caption + this);
            for (int i = 0; i <= m; i++)
                s.println(prefix + "\tat " + trace[i]);
            // 打印重叠层的省略信息
            if (framesInCommon != 0)
                s.println(prefix + "\t... " + framesInCommon + " more");
            ……

            // 递归打印内部异常
            // Print cause, if any
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, prefix, dejaVu);
        }
    }
}
```

# 附录
## 相关类及方法代码
### StackTraceElement
```java
package java.lang;

import java.util.Objects;

public final class StackTraceElement implements java.io.Serializable {
    // Normally initialized by VM (public constructor added in 1.5)
    private String declaringClass;
    private String methodName;
    private String fileName;
    private int    lineNumber;

    /**
     * 获取执行点所属源文件名称
     * @return the name of the file containing the execution point
     *         represented by this stack trace element, or {@code null} if
     *         this information is unavailable.
     */
    public String getFileName() {
        return fileName;
    }

    /**
     * 获取执行点在源文件中的行号
     * @return the line number of the source line containing the execution
     *         point represented by this stack trace element, or a negative
     *         number if this information is unavailable.
     */
    public int getLineNumber() {
        return lineNumber;
    }

    /**
     * 获取执行点所属类的全限定名
     * @return the fully qualified name of the {@code Class} containing
     *         the execution point represented by this stack trace element.
     */
    public String getClassName() {
        return declaringClass;
    }

    /**
     * 获取执行点所属方法名称
     * @return the name of the method containing the execution point
     *         represented by this stack trace element.
     */
    public String getMethodName() {
        return methodName;
    }

    public String toString() {
            return getClassName() + "." + methodName +
                (isNativeMethod() ? "(Native Method)" :
                 (fileName != null && lineNumber >= 0 ?
                  "(" + fileName + ":" + lineNumber + ")" :
                  (fileName != null ?  "("+fileName+")" : "(Unknown Source)")));
    }
}
```

### Throwable
```java
public class Throwable implements Serializable {
    
    private Throwable cause = this;
    
    public synchronized Throwable getCause() {
        return (cause==this ? null : cause);
    }
    
    protected Throwable(String message, Throwable cause,
                            boolean enableSuppression,
                            boolean writableStackTrace) {
            if (writableStackTrace) {
                fillInStackTrace();
            } else {
                stackTrace = null;
            }
            detailMessage = message;
            this.cause = cause;
            if (!enableSuppression)
                suppressedExceptions = null;
        }
    
    private StackTraceElement[] stackTrace = UNASSIGNED_STACK;
    /**
     * A shared value for an empty stack.
     */
    private static final StackTraceElement[] UNASSIGNED_STACK = new StackTraceElement[0];
    
    public synchronized Throwable fillInStackTrace() {
        if (stackTrace != null ||
            backtrace != null /* Out of protocol state */ ) {
            fillInStackTrace(0);
            stackTrace = UNASSIGNED_STACK;
        }
        return this;
    }

    public StackTraceElement[] getStackTrace() {
        return getOurStackTrace().clone();
    }
    
    private synchronized StackTraceElement[] getOurStackTrace() {
        // Initialize stack trace field with information from
        // backtrace if this is the first call to this method
        if (stackTrace == UNASSIGNED_STACK ||
            (stackTrace == null && backtrace != null) /* Out of protocol state */) {
            int depth = getStackTraceDepth();
            stackTrace = new StackTraceElement[depth];
            for (int i=0; i < depth; i++)
                stackTrace[i] = getStackTraceElement(i);
        } else if (stackTrace == null) {
            return UNASSIGNED_STACK;
        }
        return stackTrace;
    }

    public void printStackTrace(PrintStream s) {
        printStackTrace(new WrappedPrintStream(s));
    }

    private void printStackTrace(PrintStreamOrWriter s) {
        // Guard against malicious overrides of Throwable.equals by
        // using a Set with identity equality semantics.
        Set<Throwable> dejaVu =
            Collections.newSetFromMap(new IdentityHashMap<Throwable, Boolean>());
        dejaVu.add(this);

        synchronized (s.lock()) {
            // Print our stack trace
            s.println(this);
            StackTraceElement[] trace = getOurStackTrace();
            for (StackTraceElement traceElement : trace)
                s.println("\tat " + traceElement);

            // Print suppressed exceptions, if any
            for (Throwable se : getSuppressed())
                se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);

            // Print cause, if any
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
        }
    }

    private void printEnclosedStackTrace(PrintStreamOrWriter s,
                                         StackTraceElement[] enclosingTrace,
                                         String caption,
                                         String prefix,
                                         Set<Throwable> dejaVu) {
        assert Thread.holdsLock(s.lock());
        if (dejaVu.contains(this)) {
            s.println("\t[CIRCULAR REFERENCE:" + this + "]");
        } else {
            dejaVu.add(this);
            // Compute number of frames in common between this and enclosing trace
            StackTraceElement[] trace = getOurStackTrace();
            int m = trace.length - 1;
            int n = enclosingTrace.length - 1;
            while (m >= 0 && n >=0 && trace[m].equals(enclosingTrace[n])) {
                m--; n--;
            }
            int framesInCommon = trace.length - 1 - m;

            // Print our stack trace
            s.println(prefix + caption + this);
            for (int i = 0; i <= m; i++)
                s.println(prefix + "\tat " + trace[i]);
            if (framesInCommon != 0)
                s.println(prefix + "\t... " + framesInCommon + " more");

            // Print suppressed exceptions, if any
            for (Throwable se : getSuppressed())
                se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION,
                                           prefix +"\t", dejaVu);

            // Print cause, if any
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, prefix, dejaVu);
        }
    }
}
```

## 参考文献
- [Understanding and Leveraging the Java Stack Trace](https://stackify.com/java-stack-trace/)
- [Java源码分析——Throwable、Exception、Error类解析](https://blog.csdn.net/hackersuye/article/details/84193536)
- [How the Java virtual machine handles exceptions](https://www.infoworld.com/article/2076868/how-the-java-virtual-machine-handles-exceptions.html)
- [The Structure of the Java Virtual Machine](https://www.informit.com/articles/article.aspx?p=2024315&seqNum=10)
- [How does the JVM handle an exception thrown by the main method?](https://softwareengineering.stackexchange.com/questions/257174/how-does-the-jvm-handle-an-exception-thrown-by-the-main-method)
- [How the Java virtual machine handles exceptions](https://www.infoworld.com/article/2076868/how-the-java-virtual-machine-handles-exceptions.html)
- [Why do the following methods of the Throwable class need to be synchronized?](https://stackoverflow.com/questions/65650239/why-do-the-following-methods-of-the-throwable-class-need-to-be-synchronized)
- [Java中寻找main函数对应类](https://www.illidan.cn/2020/08/21/spring-deduce-main-application-class/)