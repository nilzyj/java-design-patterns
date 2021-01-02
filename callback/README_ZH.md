---
layout: pattern
title: Callback
folder: callback
permalink: /patterns/callback/
categories: Idiom
tags:
 - Reactive
---

## Intent

回调是一段可执行的代码，它作为一个参数传递给其他代码，其他代码会在某个方便的时候回调（执行）这个参数。

## 解释

真实世界的例子

> 执行任务完成后，需要通知我们。我们为执行者传递一个回调方法，等待它对我们进行回调。    

简单来说

> 回调是传递给执行者的方法，它将在定义的时刻被调用。

维基百科说

> 在计算机编程中，回调，也被称为"后调用"函数，是任何作为参数传递给其他代码的可执行代码；该其他代码被期望在给定时间回调（执行）该参数。

**程序实例**

回调是一个简单的单方法接口。

```java
public interface Callback {

  void call();
}
```

接下来我们定义一个任务，在任务执行结束后执行回调。

```java
public abstract class Task {

  final void executeWith(Callback callback) {
    execute();
    Optional.ofNullable(callback).ifPresent(Callback::call);
  }

  public abstract void execute();
}

public final class SimpleTask extends Task {

  private static final Logger LOGGER = getLogger(SimpleTask.class);

  @Override
  public void execute() {
    LOGGER.info("Perform some important activity and after call the callback method.");
  }
}
```

最后，这是我们如何执行一个任务，并在任务完成后接收回调的。

```java
    var task = new SimpleTask();
    task.executeWith(() -> LOGGER.info("I'm done now."));
```

## Class diagram

![alt text](./etc/callback.png "Callback")

## 适用性

在以下情况下使用回调模式

* when some arbitrary synchronous or asynchronous action must be performed after execution of some defined activity.

## Real world examples

* [CyclicBarrier](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CyclicBarrier.html#CyclicBarrier%28int,%20java.lang.Runnable%29) constructor can accept a callback that will be triggered every time a barrier is tripped.
