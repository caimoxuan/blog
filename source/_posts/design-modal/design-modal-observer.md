---
title: 行为型-观察者模式
date: 2019-11-12 09:47:47
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 协同协作观察者

&emsp;&emsp;观察者模式是一个使用频率比较高的模式，毕竟许多的程序都涉及到数据的改变，不可避免的就需要对这些数据的改变进行监控；特别的当一个对象的属性改变的时候，需要对其他的依赖与它的对象进行通知来进行更新（这里我认为是有一定的限制的，如这些更新操作相互无依赖，无顺序要求），观察者模式就比较合适。其次，由于观察者之间的无依赖，无顺序的要求，观察者之后的更新操作如果比较耗时的化，还可以优化成异步的，从而提高接口的效率。

<!-- more -->

### JDK中的观察者

&emsp;&emsp;jdk中早在1.0版本中就已经给我们提供了观察者的实现方式，先来看看jdk中的观察者是怎么做的：

首先是jdk中的Observer接口，它定义了观察者的规范
``` java
public interface Observer {

    //观察者中应该只有一个更新操作，当收到通知的时候来触发这个更新操作， 其中参数Observable是被观察的对象，Object是通知携带参数
    void update(Observable o, Object arg)
}
```
接下来看看被观察对象的定义
``` java
public class Observable {
    //这个状态标识是否应该通知
    private boolean changed = false;
    //这里保存观察者的列表
    private Vector<Observer> obs;
    //省略添加 、 删除观察者等方法，看最主要的方法进行通知
    public void notifyObservers(Object arg) {
        synchronized (this) {
            //这里判断了是否需要通知，所以使用的时候不要忘记改状态
            if(!changed) {
                return；
            }
            //这里把状态给改回来，之所以这里同步是怕多线程改状态造成状态不一致
            clearChanged();
        } 
        //稍微改了一下jdk的写法，这里就是循环遍历通知各个观察者；
        Observer[] arrLocal = obs.toArray();
        for (int i = arrLocal.length-1; i>=0; i--) {
            arrLocal[i].update(this, arg);
        }
    }
    //这里有一个重载的方法，如果不需要参数的化
    public void notigyObservers () {
        //可以看到实际上就传一个空参数
        notigyObservers(null);
    }
}
```
好了，这里的jdk帮助定义的观察者模式非常的简单，使用相信也不用代码演示，最简单的用法：
1. 首先将观察者实现Observer接口；
2. 被观察者继承Observable类；
3. 在被观察者对象的构造函数中addObserver(Observer)添加观察者
4. 在被观察者发生变化的时候依次调用setChanged();notifyObservers(Object);

以上就是jdk的观察者模式的使用。但是，很少会有人去使用jdk帮助定义的，为什么呢？这里我个人总结了几条：
1. Observable中有一些不通用的方法；
2. Observable中的观察者列表是一个Vector,这是一个不推荐使用的列表，而且基本使用不到他的同步功能；
3. 再通知前需要调用一次setChanged(),个人觉得这个也有一些的多此一举，一般不会需要通知状态，并且容易遗忘；

### spring中的观察者

&emsp;&emsp;当然，这样的模式在spring中也有相应的实现的方式，并且，在spring中能使用spring的依赖注入的特性，甚至都不用显示的将观察者添加到被观察者中。那么如何在spring中使用观察者模式呢：
1. 创建被观察对象继承ApplicationEvent
2. 创建观察者对象实现ApplicationListener<ApplicationEvent>并且在泛型中注明是观察哪种类型的对象，并注册到spring中
3. 使用ApplicationContext来调用publishEvent(ApplicationEvent)来发布事件

spring中的观察者模式使用可以说更加的方便了，但是这样也会有一些其他问题，比如事件的发布分布在系统的各个位置，可能会难以定位到事件的源头，所以应该注意日志的打印等。


### 实现异步观察者
&emsp;&emsp;当然，无论怎样的使用观察者模式，都不如自己熟悉的代码，如果对别人的代码不够熟悉，就可能不知道人家的一些限制和特性，自己实现自己使用才会更加的顺手和保险；如果想要spring中的观察者异步，只需要注解@Async就可以完成；
&emsp;&emsp;通过jdk的观察者可以看到，我们要实现观察者只要准备好被观察者和观察者们;其次，要让他异步，就要让他在其他线程中继续执行（或者使用mq等，这里简单实现）；

改造Observable
``` java
public class MyObservable {

    private List<Observer> obs;

    public MyObservable () {
        obs = new ArrayList<>();
    }

    //这里可以根据需要添加参数
    public void notifyObservers (Object arg) {
        for(Observer o : obs) {
            o.update(this, arg);
        }
    }

}
```
这样是一个最简单的改造，暂时把用不到的东西去除，然后利用线程池来添加异步处理；这里这个线程池是所有观察者公用的，不能每个被观察者都有一个线程池；接下来继续改造：
``` java
public class MyObservable {

    private List<Observer> obs;
    //添加线程池持有（这里线程池全局静态的话也没有问题体现一个公用）
    private ExecutorService executorService;

    public MyObservable () {
        obs = new ArrayList<>();
    }

    //添加一个传入线程池的构造方法
    public MyObservable (ExecutorService executorService) {
        obs = new ArrayList<>();
        this.executorService = executorService;
    }

    //这里将执行改成异步
    public void notifyObservers (Object arg) {
        for(Observer o : obs) {
            //存在线程池就将执行转换成异步
            if(this.executorService != null) {
                executorService.execute(() -> o.update(this, arg));
            } else {
                o.update(this, arg);
            }
            
        }
    }

}

```
这样就能异步执行任务了，但是这样做所有的观察者都在异步执行任务，想要完成spring那样对观察者方法添加注解的方式，还需要来一个自定义注解，然后注解在需要异步执行的观察者方法中，然后在这个notifyObservers方法中遍历观察者对象的时候反射查看是否注解自定义注解，最后对注解了自定义注解的方法进行异步处理。
这里就提供一种思路，我觉得程序没有完全的标准的说法，可以这样实现，也可以那样实现，只是有时候像不周到的话会造成一些意想不到的bug。另外再说一个线程池的创建规范：使用ThreadPoolExecutor的构造函数来创建线程，而不要使用Executors来创建，因为自己知道线程池如何工作的，可以避免一些bug。详细的说法参考《java开发手册-阿里巴巴编码规约》