---
title: 创建型-工厂模式
date: 2019-10-21 14:30:58
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 结构多变的工厂模式

&emsp;&emsp;工厂模式也有许多的品种， 这一度让我陷入理解的僵局。明明是这么简单的一个模式，为什么回让我有点恐惧去将这几种工厂模式好好的撸一撸... 今天要好好的做一次尝试。
&emsp;&emsp;工厂模式是一个非常常用的模式，不少的框架中都能看到它的影子： mybatis、spring、dubbo等等，其中spring的BeanFactory和ApplicationContext更是spring的核心。
&emsp;&emsp;先看看目前我了解到的工厂模式的几个品种： 简单工厂、工厂方法、抽象工厂。嗯个人就知道这三种。

<!-- more -->

### 简单工厂

&emsp;&emsp;简单工厂确实是很简单，搞一个接口Shape，有实现类Triangle和Circle。嗯可以将Shape当作模具，而实现类就是要胜场的产品。接下来找一个工厂ShapeFactory,让它通过业主给定的图纸来找到对应模具生产产品。
``` java
public class ShapeFactory {

    public Shape getShape(String drawing) {
        //按图纸返回对应产品
    }
}

```
&emsp;&emsp;个人觉得前面的模具、产品啥的都没有太大的区别，就是这个图纸有点说法，常见的图纸： 字符串表示, 枚举表示，classType表示。个人觉得用字符串表示实在是不太稳妥，毕竟现在的接口都讲究一个傻瓜式，字符串传个啥IDE都不会报错，最好还是用后面的2中表示方式：
``` java
public class ShapeFactory {

    public Shape getShape(ShapeType type) {
        //按图纸返回对应产品
    }
}

public enum ShapeType {
    CIRCLE,
    TRIANGLE;
}

```
这时规定了入参， 就避免了未知的输入参数。

使用classType作为入参，可以将工厂变得更加灵活, 直接来个最灵活的体会一下：
``` java
public <E> E getObject(Class<E> clazz) {
    return clazz.newInstance();
}
```
&emsp;&emsp;嗯确实有点灵活过分了，但是这样只适用于创建一个简单对象，这在《大话设计模式》中被称作抽象工厂的改进（反射 + 简单工厂），只不过他用的还是字符串参数，用`Class.forName()`的方式代替传入`Class<?>`。个人觉得简单工厂，用枚举的方式比较规范化。
&emsp;&emsp;简单工厂如果要添加产品， 就需要修改对应生产产品的方法，这不符合开闭原则，那么就需要进化一步。

### 工厂方法

这时工厂也有了上级总包:
``` java
public interface IFactory {
    Shape getShape();
}
```
&emsp;&emsp;特定的工厂只生产特定的产品，这就不需要图纸了，想要什么产品就去什么厂里拿，想想生活中，你想要面粉就去面粉厂是吧。不要和我说去超市，那可不是工厂。这时我想要一个Circle： 
``` java
public class CircleFactory implements IFactory {
    
    public Shape getShape(){
        return new Circle();
    }
}
```
这时我们能从这个工厂拿到想要的东西，添加生产产品的时候也能够不修改原有逻辑了。但是每添加一个产品，就多了一个工厂，编码成本提升了。可以好好衡量以下你的工厂是不是一只需要变化然后做选择。


### 抽象工厂

&emsp;&emsp;简单工厂和工厂方法中可以看出，每个工厂生产的东西比较单一。实际上一个工厂应该会具备生产一系列产品的能力，我们再次对工厂进行改进，我们之前的工厂已经能产生一个图形，现在要在图形上打上特定的logo防止假冒伪劣：
``` java
//先添加一个Logo接口
public interface Logo {
    void printLogo();
}
//Circle和Triangle各自有各自的logo
public class CricleLogo implements Logo {
    @Override
    public void printLogo() {
        System.out.println("logo Circle!");
    }
}

public class TriangleLogo implements Logo {
    @Override
    public void printLogo() {
        System.out.println("logo Triangle!");
    }
}

// 在工厂接口中添加制作logo的方法, 抽象工厂（这里个人觉得也可以还是接口）
public abstract class IFactory {
    public abstract Shape getShape();
    public abstract Logo getLogo();
}

//Circle的工厂, Triangle的也是一样的
public class CircleFactory extends IFactory {
    //要实现父类的2个抽象方法
    @Override
    public Shape getShape() {
        return new Cricle();
    }

    @Override
    public Logo getLogo() {
        return new CricleLogo();
    }
}
```
好了现在的工厂具有了生产一系列产品的能力，可有有选择性的获取shape或logo，升值可以获取2个产品的组合。这个时候可以发现， 如果去掉logo这个产品，其实他就是一个工厂方法模式。也可以这样分类工厂，将Shape分作一个工厂，Logo的制作分为一个工厂，取消一座工厂，也就变成了工厂方法。抽象工厂给一系列的产品提供了一个组合的可能，这一系列就是比较灵活的一个东西了。写法不必拘泥，但是当完成编码的时候回过头来看看，是不是和不使用模式的时候有所改进了，这可能需要时间来检验了。