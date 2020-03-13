---
title: CountDownLatch-计数阀门
date: 2019-12-13 19:28:26
thumbnail: 
categories:
    - java
tags:
    - experience
---

## JUC之CountDownLatch

说完了CyclicBarier，现在来了解一下CountDownLatch的使用，还是先说说CountDownLatch的个人理解：就类似一个考场一样，里面由考试的学生和监考的老师（可以看作一个特殊的学生），考生做考卷的时间不一样，那么交卷就会存在先后。考完一个，那么这个考生就走出教室，完成任务，教室里的学生就少了一个，直到最后一个考生离开，老师离开，那么这个教室就空闲了，就完成了任务；这里的教室就是CounDownLatch，学生就是一个个的线程，学生之间互不影响，没有说当所有考生都考完才能一起离开；

<!-- more -->

1. 构造函数

``` java

// 只有一个构造函数，就是传入计数的线程数量
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}

```

2. 常用方法

``` java
//这里还是列举常用的方法，其中的Sync 是CountDownLatch中的一个内部类，通过它来计数；

// 调用这个让之后的操作都等待CoundownLatch计数为0才执行
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}


// 将计数器较少一个
public void countDown() {
    sync.releaseShared(1);
}

...

```

3. 简单使用

我们还是使用和CyclicBarrier中相同的例子，这里使用CountDownLatch来改造一下

``` java

 public static void main(String[] args) throws InterruptedException {
    Map<String, Integer> map = new ConcurrentHashMap<>();

    CountDownLatch countDownLatch = new CountDownLatch(100);
    for(int i = 0 ; i < 100; i++) {
        new Thread(() -> {
            try {
                Integer test = map.merge("test", 1, Integer::sum);
                countDownLatch.countDown();
                // 这里不会等待
                System.out.println("mission result: " + test);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }
    // 执行等待，直到所有线程完成；
    countDownLatch.await();
    // 等待完成之后执行；
    System.out.println(map.get("test"));

}

```


4. 特点总结

CountDownLatch类似一个阀门，使用await将阀门关闭，进行等待，计数器全部跑完之后开启阀门，await之后的代码继续执行；
和CyclicBarrier的不同是，它不会让执行了`countDowm`的线程进行等待，类似执行一个放走一个，而不是大家都等到最后一个完成一起走；
其次，CyclicBarrier可以使用reset重复利用，而CountDownLatch结束等待运行完成之后就失去效果了，也就是说他是一次性用品；
CountDownLatch会在计数器不为0的时候await一直等待，在一些springboot项目中，如果不是web项目，springboot启动完成会自动关闭，所以会有使用CountDownLatch来维持springboot项目的启动的手法。在看到springboot启动类中的CountDownLatch不要觉得奇怪。