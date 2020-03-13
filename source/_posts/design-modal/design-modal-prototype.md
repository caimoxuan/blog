---
title: 创建型-原型模式
date: 2019-10-22 15:58:00
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 快速分裂原型模式

&emsp;&emsp;试想这样一个场景，你在阿里云上购买了一台云服务器，上去按装了java环境，配置了一些常用工具，使用的没什么问题。在你的苦心经营下，一台服务器的资源已经满足不了业务流量，现在你又买了一台，然后再来一次重新配置java环境...然后知道了阿里云给你提供了镜像复制...
&emsp;&emsp;原型模式就是这样，当一个对象比较复杂，重新创建一个成本会比较大，但是又需要一个新的具有完全相同属性的对象，这个时候就可以用原型模式。它具有复制一个新对象的能力，同时还能保证性能。
&emsp;&emsp;你很少去蛋糕店，每次去蛋糕店，是不是会对陈列的蛋糕样品充满好奇。样品是真的吗，这样不会浪费吗...当你没有自己DIY一个好看的蛋糕的能里，只能按照样品选一个的时候，蛋糕店也不会把样品给你，而是照着样品`复制`一个包装好给你。

<!-- more -->

``` java 
public class Cake implements Cloneable {

    @Override
    public Cake clone(){
        Cake cake = null;
        try {
            cake = (Cake) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return cake;
    }
}
```
好了，一个相当简单的原型就完成了。什么，这和你看过的原型不太一样？最主要的东西在这了，剩下的都是原型的扩展，凡事循序渐进。这里就会有一个问的比较多的问题，这里的clone是Object类中的方法，是native方法，它是浅拷贝的（这里不讨论深浅拷贝的区别）。那么要实现深拷贝，怎么做呢？
``` java
public class Cake implements Cloneable {

    //蛋糕里面有好多的水果，这也是需要复制的
    List<Fruit> fruits;
    //set get

    @Override
    public Cake clone(){
        Cake cake = null;
        try {
            cake = (Cake) super.clone();
            //第一种：手动复制 可以发现List已经实现了Cloneable，否则内部也要实现Cloneable
            cake.setFruit(this.getFruit().clone());
            //第二种： 通过Serializable序列化后读取对象流，例如commons-lang3中的工具类 必须实现Serializable接口才行，包阔内部的对象（本例中的Fruit）
            cake = SerializationUtils.clone(this);
            //第三种： 转json后反序列化回对象，借助fastjson\gson\jackson等工具
            Cake cake = JSON.parseObject(JSON.toJSONString(this), Cake.class);

        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return cake;
    }
}
```
我想应该就是原型模式的核心了，至于如何扩展就要看各自的业务如何了。这个时候店员把复制好的蛋糕给你，你可以回去吃了它，但是放在那里的样品却不曾改变。如果你发现你对你复制的对象的操作能够影响到原来的对象，那一定是出了什么问题...