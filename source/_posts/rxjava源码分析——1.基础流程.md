---
title: rxjava源码分析——1.基础流程
date: 2020-06-09 11:01:54
tags:
    - java
    - rxjava
description: rxjava作为一个异步框架blabla
---
# demo
这里以rxjava1.2.0为例子进行分析，首先写一个最简单的demo之后我们逐步debug走一下它的运行流程。
```java
import rx.Observable;
import rx.Observer;
import rx.Subscriber;

public class RxjavaTest {
    public static void main(String[] args) {
        Observer<String> observer = new Observer<String>() {

            public void onCompleted() {
                System.out.println("\nobserver receive onCompleted\n");
            }

            public void onError(Throwable e) {
                System.out.format("\nobserver receive onError:\n\tmessage: %s\n\tstackTrace: \n", e.getMessage());
                e.printStackTrace();
            }

            public void onNext(String source) {
                System.out.format("\nobserver receive onNext:\n\tmessage: %s\n", source);
            }
        };

        Observable observable = Observable.create(new Observable.OnSubscribe() {
            public void call(Object o) {
                Subscriber subscriber = (Subscriber) o;
                subscriber.onNext("observable send message1");
                subscriber.onNext("observable send message2");
                subscriber.onError(new RuntimeException("here is an exception"));
                subscriber.onCompleted();
            }
        });

        observable.subscribe(observer);
    }
}

```
这里observable有三个方法
- onNext():发送一条消息
- onError():发送一条错误信息
- onCompleted():告知observer消息发送完毕

消息发送过程中onError或onComplete触发后本次消息发送流程结束，后边的消息不会再发送，包括后边所有的onNext、onError和onCompleted。
demo中如果想触发onCompleted需要注释掉onError。

# 流程分析
# 入口
`observable.subscribe(observer);`触发了observable发送消息，上边的代码都是在构造observer和observable。
## subscribe()
observable.subscribe方法有六个重载方法，但最后都可以归结到一个方法

1. 无参数
    ```java
    public final Subscription subscribe() {
        Action1<T> onNext = Actions.empty();
        Action1<Throwable> onError = InternalObservableUtils.ERROR_NOT_IMPLEMENTED;
        Action0 onCompleted = Actions.empty();
        return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
    }
    ```

2. 传入onNext
    ```java
    public final Subscription subscribe(final Action1<? super T> onNext) {
           if (onNext == null) {
               throw new IllegalArgumentException("onNext can not be null");
           }
    
           Action1<Throwable> onError = InternalObservableUtils.ERROR_NOT_IMPLEMENTED;
           Action0 onCompleted = Actions.empty();
           return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
    }
    ```
   
3. 传入onNext、onError
    ```java
    public final Subscription subscribe(final Action1<? super T> onNext, final Action1<Throwable> onError) {
        if (onNext == null) {
            throw new IllegalArgumentException("onNext can not be null");
        }
        if (onError == null) {
            throw new IllegalArgumentException("onError can not be null");
        }

        Action0 onCompleted = Actions.empty();
        return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
    }
    ```
    
4. 传入onNext、onError、onCompleted
    ```java
    public final Subscription subscribe(final Action1<? super T> onNext, final Action1<Throwable> onError, final Action0 onCompleted) {
        if (onNext == null) {
            throw new IllegalArgumentException("onNext can not be null");
        }
        if (onError == null) {
            throw new IllegalArgumentException("onError can not be null");
        }
        if (onCompleted == null) {
            throw new IllegalArgumentException("onComplete can not be null");
        }

        return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
    }   
    ```
   
5. 传入Observer
    ```java
   public final Subscription subscribe(final Observer<? super T> observer) {
       if (observer instanceof Subscriber) {
           return subscribe((Subscriber<? super T>)observer);
       }
       if (observer == null) {
           throw new NullPointerException("observer is null");
       }
       return subscribe(new ObserverSubscriber<T>(observer));
   }
    ```
   
可以看到上边的五个方法是对不同的入参封装、补充了onNext、onError、onCompleted三个方法，最终都是调用下边的这个方法
```java
public final Subscription subscribe(final Observer<? super T> observer) {
  if (observer instanceof Subscriber) {
    return subscribe((Subscriber<? super T>)observer);
  }
  if (observer == null) {
    throw new NullPointerException("observer is null");
  }
  return subscribe(new ObserverSubscriber<T>(observer));
}
```
这个方法中将observer封装成了subscriber，subscriber中多了一些方法。

*TODO:* subscriber&observer

之后调用static的sbscribe方法并传入当前observable(代码中的this)
```java
public final Subscription subscribe(Subscriber<? super T> subscriber) {
        return Observable.subscribe(subscriber, this);
}
```
最后调用到核心的subscribe方法，如下：
```java
static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {
     // validate and proceed
        if (subscriber == null) {
            throw new IllegalArgumentException("subscriber can not be null");
        }
        if (observable.onSubscribe == null) {
            throw new IllegalStateException("onSubscribe function can not be null.");
            /*
             * the subscribe function can also be overridden but generally that's not the appropriate approach
             * so I won't mention that in the exception
             */
        }

        // new Subscriber so onStart it
        subscriber.onStart();

        /*
         * See https://github.com/ReactiveX/RxJava/issues/216 for discussion on "Guideline 6.4: Protect calls
         * to user code from within an Observer"
         */
        // if not already wrapped
        if (!(subscriber instanceof SafeSubscriber)) {
            // assign to `observer` so we return the protected version
            subscriber = new SafeSubscriber<T>(subscriber);
        }

        // The code below is exactly the same an unsafeSubscribe but not used because it would
        // add a significant depth to already huge call stacks.
        try {
            // allow the hook to intercept and/or decorate
            RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);
            return RxJavaHooks.onObservableReturn(subscriber);
        } catch (Throwable e) {
            // special handling for certain Throwable/Error/Exception types
            Exceptions.throwIfFatal(e);
            // in case the subscriber can't listen to exceptions anymore
            if (subscriber.isUnsubscribed()) {
                RxJavaHooks.onError(RxJavaHooks.onObservableError(e));
            } else {
                // if an unhandled error occurs executing the onSubscribe we will propagate it
                try {
                    subscriber.onError(RxJavaHooks.onObservableError(e));
                } catch (Throwable e2) {
                    Exceptions.throwIfFatal(e2);
                    // if this happens it means the onError itself failed (perhaps an invalid function implementation)
                    // so we are unable to propagate the error correctly and will just throw
                    RuntimeException r = new OnErrorFailedException("Error occurred attempting to subscribe [" + e.getMessage() + "] and then again while trying to pass to onError.", e2);
                    // TODO could the hook be the cause of the error in the on error handling.
                    RxJavaHooks.onObservableError(r);
                    // TODO why aren't we throwing the hook's return value.
                    throw r; // NOPMD
                }
            }
            return Subscriptions.unsubscribed();
        }
    }
```
这里去除掉检查异常部分主要有三个过程:
1. 调取`subscriber.onStart();`这个方法是subscriber比observer多的方法，在核心方法开始之前触发。
这里因为demo中用的是observer所以这个方法中并没有任何操作。

2. subscriber包装成SafeSubscriber
    ```java
    if (!(subscriber instanceof SafeSubscriber)) {
        // assign to `observer` so we return the protected version
        subscriber = new SafeSubscriber<T>(subscriber);
    }
   ```
3. 触发核心方法
   ```java
   RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);
   return RxJavaHooks.onObservableReturn(subscriber);
    ```
`RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);`
的前半部分`RxJavaHooks.onObservableStart(observable, observable.onSubscribe)`返回的就是经过封装和处理的业务中的observable，后边的call方法即为调用业务逻辑中的相关call方法。

## RxJavaHooks
通过上边的分析我们可以看到核心的方法是通过RxJavaHooks触发的，下边我们会分析到底具体挂了哪些钩子都做了什么，具体怎么触发的。

首先我们要明白这里传入的observable就是经过多层转化后业务代码中的observable

我们先来看下`observable.onSubscribe`中的`onSubscribe`，这里只有一个赋值的地方就是
```java
protected Observable(OnSubscribe<T> f) {
    this.onSubscribe = f;
}
```
而这个方法也只有一个调用方法
```java
public static <T> Observable<T> create(OnSubscribe<T> f) {
    return new Observable<T>(RxJavaHooks.onCreate(f));
}
```
这里就和业务代码联系起来了，也就是说这里的`onSubscribe`是业务代码中调用`Observable.create`方法时传入的`onSubscribe`。
到这通过属性名大概也知道为什么在subscribe时会调用call方法中的操作了，核心是调用了onSubscribe，当然是在subscribe时候调用23333。

*TODO:* 所以消息每次在subscribe时候推送过来，是pull的还是push的...毕竟也没看到可以往里边补充信息

我们可以看到`RxJavaHooks.onObservableStart(observable, observable.onSubscribe)`返回的也是一个`onSubscribe`，后边的`call()`方法调用具体的业务逻辑。

```java
public static <T> Observable.OnSubscribe<T> onObservableStart(Observable<T> instance, Observable.OnSubscribe<T> onSubscribe) {
    Func2<Observable, Observable.OnSubscribe, Observable.OnSubscribe> f = onObservableStart;
    if (f != null) {
        return f.call(instance, onSubscribe);
    }
    return onSubscribe;
}
```
这里`Func2<Observable, Observable.OnSubscribe, Observable.OnSubscribe> f`是一个入参`Observable, Observable.OnSubscribe`，出参`Observable.OnSubscribe`的变换方法，整个方法的作用在于，如果这里挂了`onObservableStart`方法，那么在执行onSubscribe的call方法之前需要先对其进行加工变换。

RxJavaHooks类中静态方法调用了init方法，而init方法中定义了各个钩子
```java
static {
    init();
}

static void init() {
    onError = new Action1<Throwable>() {
        @Override
        public void call(Throwable e) {
            RxJavaPlugins.getInstance().getErrorHandler().handleError(e);
        }
    };

    onObservableStart = new Func2<Observable, Observable.OnSubscribe, Observable.OnSubscribe>() {
        @Override
        public Observable.OnSubscribe call(Observable t1, Observable.OnSubscribe t2) {
            return RxJavaPlugins.getInstance().getObservableExecutionHook().onSubscribeStart(t1, t2);
        }
    };

    onObservableReturn = new Func1<Subscription, Subscription>() {
        @Override
        public Subscription call(Subscription f) {
            return RxJavaPlugins.getInstance().getObservableExecutionHook().onSubscribeReturn(f);
        }
    };
    ...
}
```
这里`onObservableStart`钩子最终执行就是原样返回了`onsubScribe`给外层，所以最终在上文所述的核心代码处调用了onSubscribe.call()，从而调用了业务代码，在observable中调用了onNext、onError、onCompleted发送消息。

## onNext、onError、onCompleted
在observable执行`subscriber.onNext("observable send message1");`发送消息的过程中可以通过debug跟进去看到他调用了SafeSubscriber的onNext方法
```java
@Override
public void onNext(T args) {
    try {
        if (!done) {
            actual.onNext(args);
        }
    } catch (Throwable e) {
        // we handle here instead of another method so we don't add stacks to the frame
        // which can prevent it from being able to handle StackOverflow
        Exceptions.throwOrReport(e, this);
    }
}
```
而这里的 调用了`ObserverSubscriber.OnNext`
```java
@Override
public void onNext(T t) {
    observer.onNext(t);
}
```
从而最终调用到了业务代码中Observer的相关逻辑。

调用逻辑理清了，那么我们来看下是什么时候把observer封装进去的。

上文我们提到了在observer的核心subscribe方法中做了三件事，其中第二件事就是将传入的observer封装成一个SafeSubscriber，
也就是执行了如下方法
```java
public SafeSubscriber(Subscriber<? super T> actual) {
    super(actual);
    this.actual = actual;
}
```
而ObserverSubscriber那一层是在执行
可以看subscribe方法过程中封装的，可以查阅上文重载的subscribe方法中的第五个，传入Observer。可以看到最后封装了一层`return subscribe(new ObserverSubscriber<T>(observer));`
因此这里的调用逻辑从外到里变成了SafeSubscriber->ObserverSubscriber->业务observer中的onNext方法。