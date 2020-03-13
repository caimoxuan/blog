---
title: 创建型-单例模式
date: 2019-10-21 11:29:31
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 真的了解单例吗

&emsp;&emsp;单例模式一般都是设计模式介绍中的第一个，属于创建型模式，看过大多数的介绍都在说单例模式是最简单的模式之一，但我觉得不是这样的。这一个设计模式，真的涉及太多的知识点以及原理，要说简单的话，就只有他的类图比较简单吧，除非对自己的要求就一个饿汉式就完了。
&emsp;&emsp;单例的好处是减少了内存的开销，避免了能实现相同功能的类重复的创建和销毁，而只初始化一次，完成多次的任务。

<!-- more -->

### 单例模式-饿汉式

``` java
public class Singleton {

    private Singleton instance = new Singleton();

    public static Singleton getInstance() {
        return instance;
    }

}
```
饿汉式单例真的非常的简单，如果单例只有一个模式，却是算得上最简单的模式之一。但是饿汉式单例被认为有个缺陷，这样的类放入项目中，不论你是否会使用它，它的对象在类加载的时候就会存在，这样就造成了内存的浪费，万一一直没有用到它呢。其实我觉得一般的对象没那么大，这样其实影响并不是很大。

### 单例模式-懒汉式

懒汉式单例有很多的写法，虽然代码没有几行，但是步步杀机！
 
``` java
public class Singleton {
    private Singleton instance;

    public static Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```
&emsp;&emsp;这样写也能实现懒加载，但是在多线程访问的时候就会有隐患。甚至严格来说它不是单例模式，因为他可能产生多个实例（如何产生就不多说了，应该没有不知道的）。
&emsp;&emsp;那么如何来改进呢？首先考虑的是线程安全。实现安全，那就是加锁。修改`getInstance()`方法：

``` java
public static synchronized Singleton getInstance() {
    if(instance == null) {
        instance = new Singleton();
    }
    return instance;
}

```

好了，这下看似线程安全了，但是又出了一个问题。判断锁是有代价的，而观察这个方法，貌似只有在instance还没有初始化的时候才需要判断锁，而在这之后对这个锁的判断都是多此一举，白白降低了这个方法的效率。接下来如何解决呢，就是double check, 继续修改`getInstance()`方法：
``` java
public static Singleton getInstance() {
    if(instance == null) {
        synchronized (Singleton.class) {
            if(instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```
同步代码和上面的方法一样，想解决线程安全的问题。在同步代码外加了一层判空，试想只有当instace还没有初始化的时候才会进入同步代码中，而instance初始化完成之后，调用这个方法都不会再判断锁了，这就解决了方法效率问题，那么这样真的就大功告成了吗？其实还没有...
``` java
public class Singleton {
    private volatile Singleton instance;

    public static Singleton getInstance() {
        if(instance == null) {
            synchronized (Singleton.class) {
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

```
在上面的代码中，可以算是大功告成了。可以看到再声明instance的时候加了一个`volatile`，他的作用是什么，这里只能说它防止了虚拟机对`instance = new Singleton()`这段代码的指令重排，并且在多线程的情况下保证了各个线程对于instance的可见性，这里不扩展。很喜欢问的一个面试题： `volatile`和`synchronized`有什么异同。我觉得了解了这个单例为什么要这么写，就已经能回答了。

### 静态内部类

很多的地方都只会介绍以上2种形式的单例，对以下者2种介绍的会比较少。
``` java
public class Singleton {

    private static class SingleHolder {
        public static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingleHolder.INSTANCE;
    }

}
```
这种方式同样是懒加载，由jvm来保证初始化一次，这里又涉及到类加载机制中的初始化过程。这种方式使用的也比较少，在单例对象比较简单的时候可以使用。

### 枚举

这是Effective Java 作者 Josh Bloch所提倡的方式，可以绝对的保证初始化一次，并且可以支持类的序列化，我也觉得这很简单，但是这也不是懒加载的。
``` java
public enum Singleton {
    INSTANCE;
}
```
嗯就这样，然后将单例锁提供的方法都写好就能用了:
``` java
public enum Singleton {
    INSTANCE;
    public void method () {
        System.out.println("method");
    }
}
```
如何使用呢： 
``` java
public class Test {
    public static void main(String[] args) {
        Singleton.INSTANCE.method();    
    }
}
```
为什么这种方式用的比较少呢，由于java1.5才出现的enum，大多的教程都介绍的前两种单例模式，而枚举在实际中用的比较少，更不要说用它来实现单例了。

总之使用上述单例模式的最终形态我觉得都没有什么问题，影响都不大，也许对各人来说都是看的优雅不优雅吧，真的因为单例的实现方式造成了比较大的影响，那么我觉得首先应该看看这个单例里面的东西是否应该放在代码里面吧。