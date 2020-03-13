---
title: CyclicBarrier-回环栅栏的使用
date: 2019-12-13 21:22:41
thumbnail: 
categories:
    - java
tags:
    - experience
---

## JUC之CyclicBarrier

CountDownLatch 、 CyclicBarrier 、Semaphore这三个java提供的工具经常放在一起做比较，接下来三篇分别使用它们来区别他们的不同；先说说CyclicBarrier的个人理解，就像一个栅栏一样，举行赛跑的时候，所有的人必须全都到白线前等待，然后才开始；这里的白线就是CyclicBarrier，人就是线程；

<!-- more -->

1. 构造函数

首先看构造函数：
``` java
/**
 * @Param parties : 需要计数的Thread数量，达到数量之后栅栏开放
 * @Param barrierAction : 栅栏开放后触发的动作 
 */
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

// 这是个没有动作的栅栏
public CyclicBarrier(int parties) {
    this(parties, null);
}
```

2. 常用方法

要了解如何用，需要看它的方法, 不分析原理看 public 的方法就行了；这里只看要用到的
``` java
// 调用这个方法，说明到达栅栏，会将到达线程的计数器加1
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

其他还有一些方法，比如:
限制时间的 await
重置栅栏的 reset
获取等待线程数的 getNumberWaiting
查看栅栏开放状态的 isBroken

```

这里简单使用的话使用`await`方法就够了；


3. 简单案例

我们构造一个简单场景，多线程统计单词出现的次数， 模拟一堆线程对数据操作全部完成之后在输出结果
``` java

public static void main(String[] args) {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    //创建一个100个计数的栅栏， 并且栅栏开放之后的方法是打印最终结果
    CyclicBarrier cyclicBarrier = new CyclicBarrier(100, () -> {
        System.out.println(map.get("test"));
    });
    // 100 线程操作map
    for(int i = 0 ; i < 100; i++) {
        new Thread(() -> {
            try {
                // 这里我们就模拟对 `test` 这个单词的累加次数统计 这里使用了map的merge方法，这个方法是接口的默认方法，但是在concurrentHashMap中进行了重写， 可以看到他也是线程安全的； 
                Integer test = map.merge("test", 1, Integer::sum);
                // 每个线程操作完毕，调用栅栏的await
                cyclicBarrier.await();
                // 当栅栏开放这里才会执行
                System.out.println("mission result: " + test);
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }).start();
    }

}
```


那么我们的CyclicBarrier在这里的作用是什么呢？其实这个在多线程单元测试的时候很常见，应为这里是多线程的，在线程start完成之后，直接输出`map.get("test")`的值的话，这个时候线程是没有运行完成的，用了这个回环栅栏，那么就能让最后的输出结果在线程全部完成之后在进行；
这个效果使用CountDownLatch也可以十分方便的实现

4. 特点总结

1. 可以看见回环栅栏可以让执行线程全部等待，到所有线程都完成并调用await的时候， 栅栏开放，所有的线程一起触发之后的流程；
2. 回环栅栏那么回环体现在哪里呢？我们可以通过reset方法将它重置，这样我们就能重新使用它了；关于reset其中线程的处理可以看reset方法中的处理；