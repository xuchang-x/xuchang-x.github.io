---
title: rxjava源码分析——2.scheduler实现线程切换
date: 2020-06-10 10:37:38
tags:
    - java
    - rxjava
description: rxjava最重要的功能是他异步工作的能力，基于上一篇文章对rxjava基本调用流程的分析，本文对rxjava异步工作流程进行探讨。
---
同上篇一样，我们这里先给出一个demo，后文基于这个demo进行分析。

# demo
```java
package article;

import rx.Observable;
import rx.schedulers.Schedulers;

public class AsyncTest {

    private static volatile int observerId = 1;

    public static void main(String[] args) {
        Observable<String> observable1 = buildTimeConsumingObserver(2000);
        Observable<String> observable2 = buildTimeConsumingObserver(3000);
        
        Long start = System.currentTimeMillis();
        observable1.subscribe(onNext-> System.out.format("\nobserver receive onNext:\n\tmessage: %s\n", onNext));
        observable2.subscribe(onNext-> System.out.format("\nobserver receive onNext:\n\tmessage: %s\n", onNext));
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Long end = System.currentTimeMillis();
        System.out.format("用时%dms", end - start);
    }

    private static Observable<String> buildTimeConsumingObserver(int waitMs) {
        int id = observerId++;
        return Observable.just(1).observeOn(Schedulers.newThread()).map(integer -> {
            System.out.println("observable " + id + " begin");
            try {
                Thread.sleep(waitMs);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("observable " + id + " finish");
            return "observable " + id + " send message2, waited " + waitMs + " ms";
        });
    }
}
```
运行结果如下：
```text
observable 1 begin
observable 2 begin
observable 1 finish

observer receive onNext:
	message: observable 1 send message2, waited 2000 ms
observable 2 finish

observer receive onNext:
	message: observable 2 send message2, waited 3000 ms

observer receive onCompleted
用时3029ms
```
首先可以看到这里共用了3s左右，小于串行的5s，说明两个任务是在不同线程运行的了。

下边说明一些问题：
1. 切换线程的两个方法`observeOn()`、`subscribeOn()`是observable的实例方法而非类方法，这里的考虑在于方便后续多次切换线程时的链式调用，但同时也导致了一个问题就是没办法在`Observable.create()`之前指定scheduler，所以这里先通过`just()`方法创建了一个observable，又通过他的实例方法进一步做了耗时操作。

    - `just()`方法本身底层就是create一个observable并发出相关消息，这个在其他的文章中再做探讨。

2. main方法中的`Thread.sleep(3000);`主要是为了等新启动的两个线程输出结果后主线程再结束，这里如果不等的话会出现主线程已经结束了，子线程还没完成任务，这时候在控制台就看不到完整的输出。

3. 这里main方法中的`Thread.sleep(3000);`可以换一种写法
    ```java
    Long start = System.currentTimeMillis();
    Observable.merge(observable1, observable2).toBlocking().subscribe(onNext -> System.out.format("\nobserver receive onNext:\n\tmessage: %s\n", onNext),
            onError -> {
                System.out.format("\nobserver receive onError:\n\tmessage: %s\n\tstackTrace: \n", onError.getMessage());
                onError.printStackTrace();
            }, () -> System.out.format("\nobserver receive onCompleted\n"));
    Long end = System.currentTimeMillis();
    ```
   - 首先调用`merge()`方法将两个Observable合并成一个，这个过程中并不影响他们`observeOn()`方法在新启动线程上完成的操作。`merge()`方法本身并不在这里讨论，会再其他文章中再做说明。
   
   - `toBlocking()`将一个普通的Observable转化成BlockingObservable，内部使用CountDownLatch实现阻塞操作。

# observeOn切换线程
这里`observeOn(Schedulers.newThread())`方法分两部分

## Schedulers.newThread()
```java
public static Scheduler newThread() {
    return RxJavaHooks.onNewThreadScheduler(getInstance().newThreadScheduler);
}
```
这里进入Schedulers类中可以看到
```java
public final class Schedulers {

    private final Scheduler computationScheduler;
    private final Scheduler ioScheduler;
    private final Scheduler newThreadScheduler;

    ...
}
```
这里分三种线程调度器，计算型、io型和启动新线程类型，这里对scheduler的细节暂时不做讨论，这里引用的是`newThreadScheduler`

## observeOn
点击进入`observeOn()`方法可以看到该方法也有四个重载方法，这里调用的是
```java
public final Observable<T> observeOn(Scheduler scheduler) {
    return observeOn(scheduler, RxRingBuffer.SIZE);
}
```
其他两个分别是带有delayError、bufferSize参数的，最终都调用了
```java
public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
    if (this instanceof ScalarSynchronousObservable) {
        return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
    }
    return lift(new OperatorObserveOn<T>(scheduler, delayError, bufferSize));
}
```
当涉及到`delayError`时会调用`lift()`方法进行变换操作，当前是一个单纯发送消息的Observer，走了`instanceof ScalarSynchronousObservable`分支。

*TODO:* 后续看多种subject过程中应该会涉及到这里的`lift()`方法，后续补充。

当前分支调用方法核心如下
```java
public Observable<T> scalarScheduleOn(final Scheduler scheduler) {
    Func1<Action0, Subscription> onSchedule;

    ...

    onSchedule = new Func1<Action0, Subscription>() {
        @Override
        public Subscription call(final Action0 a) {
            final Scheduler.Worker w = scheduler.createWorker();
            w.schedule(new Action0() {
                @Override
                public void call() {
                    try {
                        a.call();
                    } finally {
                        w.unsubscribe();
                    }
                }
            });
            return w;
        }
    };

    return create(new ScalarAsyncOnSubscribe<T>(t, onSchedule));
}
```
这里最后调用的`create()`方法就是[基本调用流程](https://xuchang-x.github.io/2020/06/09/rxjava%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E2%80%94%E2%80%941-%E5%9F%BA%E6%9C%AC%E8%B0%83%E7%94%A8%E6%B5%81%E7%A8%8B/)中介绍业务代码中的`Observable.create()`方法，使用最终包装好的`ScalarAsyncOnSubscribe`创建Observable，回归到基本流程上。

而其中的包装过程分为两部分：
1. 创建`onSchedule`，上述代码的`call()`方法(下称做封装call方法)中可以看到其将业务中的`onSubscribe.call()`方法(下称作业务call方法)封装在了`onSchedule.call`中，而封装的call方法通过`scheduler.createWorker()`这个线程中调用。而具体执行线程根据策略判断，这里我们scheduler选用的newThread策略，所以call方法会被封装到一个新线程中去调用。

2. `new ScalarAsyncOnSubscribe<T>(t, onSchedule)`将`onSchedule`再次封装成一个`onObservable`

因此这里再调用通过如下逻辑onSchedule->根据`scheduler.createWorker()`策略选择调用线程->调用worker中的`call()`方法->`调用onObservable.call()`中的业务代码->触发`observer.call()`。

*TODO:* 这里调用过程中走的肯定和单纯的observable不一样，后续还需要细化

# subscribeOn

`subscribeOn()`方法和`observeOn()`方法类似，都用于控制线程，不同的是`observeOn()`控制的是`observable`的工作线程，而`subscribeOn()`是控制的`observer`的工作线程。

同时，两个方法控制的范围不同`observeOn()`控制调用链接在该方法后的方法，而`subscribeOn()`控制其之前的一段调用方法。

observeOn：
![observeOn作用范围](observeOn作用范围.jpg)
subscribeOn：
![subscribeOn作用范围](subscribeOn作用范围.jpg)

根据上边的特性，`observeOn()`两个连续调用以后边的为准，`subscribeOn()`两个连续调用以前边的为准。

![多个作用范围](多个作用范围.jpg)
