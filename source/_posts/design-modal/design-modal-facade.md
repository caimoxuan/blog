---
title: 结构型-外观模式
date: 2019-11-25 17:41:52
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---


## 防呆设计外观模式

&emsp;&emsp;外观模式对于现在的系统来说也是一个大量使用的模式，只不过用的太多了可能都没太注意倒它。我们提倡面向接口编程，因为接口提供了一种规范，让开发人员按照接口开发，而使用者按照这个接口调用。调用者不用关心接口之内是如何实现的，任何的接口问题都需要开发者来解决。就像一台计算机，我们使用计算机的时候，只用使用计算机的外设（键盘、鼠标、显示器）。我们不用关心键盘和鼠标的操作如何反应到显示器的。

<!-- more --> 

&emsp;&emsp;usb的插口的设计是有正反面的，差错了就插不进去（用点力也许也行...），这样的设计就是防呆设计，让使用者看了就知道插口怎么用，就算用错了也差不进去。配过电脑的应该知道，计算机电源的那一堆针脚，不是一个装机老司机肯定乱不清楚，但是现在都是几个几个一组，插错了就能发现。外观模式就是这样，帮助我们屏蔽了内部的实现，只暴露了使用的方式。这个很简单，我觉得它贴近一种约定。
下面我们模拟一下当按下机箱电源键的那一刻，机箱帮我们做了哪些事情
``` java
// 计算机元件
public interface Element {

    // 源键工作 voltage ： 所需电压
    void work(Integer voltage);
}

// cpu 和 主板是一起的
public class CPU implements Element {

    @Override
    public void work(Intgeer voltage) {
        if (Integer.valueOf(12).equals(volage)) {
            System.out.print("cup start");
        }
    }
}

// 固态硬盘 2.5的SATA接口固态硬盘是5v
public class SSD implements Element {

    @Override
    public void work (Integer voltage) {
        if(Integer.valueOf(5).equals(voltage)) {
            System.out.println("ssd work")
        }
    }
}


// 显卡 大多是也是12V
public class GPU implements Element {

    @Override
    public void work(Integer voltage) {
        if(Integer.valueOf(12).equals(voltage)) {
            System.out.println("gpu work");
        }
    }

}
// 还有其他的...不举例了

//接下来是机箱 提供外观的关键
@Data
public class Crate {

    private Element CPU;

    private Element GPU;

    private Element SSD;


    public void start() {
        if(this.CUP != null) {
            this.CUP.work(12);
        }
        if(this.SSD != null) {
            this.SSD.work(5);
        }
        if(this.GPU != null) {
            this.GPU.work(12);
        }
    }


}

```
由于台式机帮我们装好了机箱的，由装机人员给我们提供了接口（一个电源键），当我们按下电源键的时候
``` java
public static void main(String[] args) {
    // 首先装机人员帮我们装好了机箱
    Crate c = new Crate();
    c.setCPU(new CUP());
    c.setGPU(new GPU());
    c.setSSD(new SSD());

    // 把机箱交给我们使用
    c.start();
}
```
机箱帮助我们屏蔽了内部的复杂的元件的启动策略，我们只需要用机箱就能完成对台式机的操作。对于接口来说也是类似的，我们在调用支付宝开放平台的接口的时候，我们不用关心它内部如何实现的，只要调用他的接口完成相应的功能就行了。其实也可以这样想，所有对外部的接口，使用的都是外观的理念，具体到编码中，将复杂的操作，封装到一个操作中，让使用者能够最方便的使用，这就类似接口的防呆设计。总不能把一个发起订单的接口替换成 重复订单查询、创建订单、创建红包订单、等接口吧。

