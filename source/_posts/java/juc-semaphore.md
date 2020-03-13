---
title: Semaphore-信号量的使用
date: 2019-12-16 23:17:48
thumbnail: 
categories:
    - java
tags:
    - experience
---

## JUC之Semaphore

在juc提供的工具中，Semaphore算是比较不同的了，还是比较好区分的；还是先说说个人理解：信号量类似一个理发店，门店里的技师是有限的，有多少个技师，就能并行理多少个人；技师理完一个，就空闲了，就能为下一个顾客理发；这里的技师就相当于Semaphore中的信号，顾客就是线程；而顾客源源不断，信号要等待到有空闲的时候才能处理；可以发现如果做简单限流，信号量可以按并行线程数量来限流；

<!-- more -->

1. 构造函数

``` java
/**
 * @Param permits 许可 信号数量（同时并行线程数量）
 * @Param fair 是否公平 （公平就是会按先后顺序，先到先得，后到排队）
 */
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

// 默认这个是不公平的
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

```

2. 常用方法

``` java

// 尝试获取一个许可信号，获取不到阻塞 
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

// 释放一个许可
public void release() {
    sync.releaseShared(1);
}

// 上面的2个方法一般组合使用，其实和Lock的lock 和 unlock一样；上面都是获取1个许可，也提供获取多个的方法；

```

3. 简单使用

要使用Semaphore来实现CountDownLatch和CyclicBarrire所例举的例子是不太方便的，因为它没有提供总体的流程控制，但是为了展示区别，我们可以使用其他的方式让他达到操作完成数据之后（所有线程完成之后）再获取数据看看结果如何：

``` java

public static void main(String[] args) throws InterruptedException {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    // 这里我们控制并行线程10 并且使用公平模式 这里的线程不能大于等于100
    Semaphore semaphore = new Semaphore(10, true);
    for(int i = 0 ; i < 100; i++) {
        int finalI = i;
        new Thread(() -> {
            try {
                semaphore.acquire();
                Integer test = map.merge("test", 1, Integer::sum);
                //Thread.sleep(300);
                System.out.println("mission result: " + test);
                // 由于是按顺序的执行，这里发生的时候线程应该都执行完了
                if(finalI == 99) {
                    System.out.println(map.get("test"));
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                // 在这里防止释放中途异常
                semaphore.release();
            }
        }).start();
    }
}

```

上面的执行并不一定保证结果正确，因为这里的线程分配动作是在循环中进行的，如果线程创建并启动的速度 < 判断公平锁的速度，那么结果就会有偏差。

4. 特点总结

可以看出Semaphore并不适合做最后结果的统计，它的主要作用是控制并行线程，想让并行控制的效果更加的明显，可以在线程中sleep， 或者将Semaphore的许可信号设置为1，即使是100个线程，也能看到它能将并行线程控制成1；