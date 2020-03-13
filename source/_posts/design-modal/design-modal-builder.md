---
title: 创建型-建造者模式
date: 2019-10-22 14:24:38
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 灵活创建复杂对象的建造者

&emsp;&emsp;在大城市里，现在开发商售卖的房子，一般都是精装交付的。现在假设房子是一个对象，直接一个精装的房子交付给你，拎包入职，想想是不是很舒服。但是可能遇到了无良开发商，装修材料偷工减料，家电家具品质不佳，问题就出现了。
&emsp;&emsp;像房子这样复杂的对象，可能会有一些需求，有的提供精装交付，有的交付一个毛胚自己装修。而房子里面的这些配套是基本不变的： 冰箱、洗衣机、空调、电视等等。这时候我们就能实现一个家的建造者。

<!-- more -->

``` java
public class Home {
    //声明家里面可能或出现的家具
    private TV tv;
    
    private AirCondition ac;

    private Fridge fd;

    ......
}

public abstract class AbstractHomeBuilder {

    protected Home home; 

    public AbstractHomeBuilder() {
        //装修的时候你得有个房，至少也要有个毛胚吧；没房怎么装...
        //这里简单 new 一个， 也可以构造喊出穿进来；
        this.home = new Home();
    }


    //建造者的建造方法最后需要返回本身，你请人家配置完一件家具，要把房子交还回来吧。这样才能继续装修；
    //规定了添加这些家具的方法：
    public AbstractHomeBuilder build(TV tv) {
        this.tv = tv;
        return this;
    }

    public AbstractHomeBuilder build(AirCondition ac) {
        this.ac = ac;
        return this;
    }

    public AbstractHomeBuilder build(Fridge fd) {
        this.fd = fd;
        return this;
    }

    .....
    //装修完成交还房子；
    public Home build() {
        return this.home;
    }
}

```
好了，建造的规范有了，你可以请一个装修公司，按照上面的规格来配置家具，当然，现在都喜欢牌子，有牌子至少质量有保障，嗯，你请了一个使用品牌的装修公司：
``` java
public class HomeBuilder extends AbstractHomeBuilder {

    //Haier的装修公司给你配置的肯定都是Haier的家具；
    //这里的返回可以回头再看看里氏替换中的说法；
    //这里的重写可能在补充一些操作才需要，这里是使例子完整
    @Override
    public HomeBuilder build(TV tv) {
        this.tv = tv;
        return this;
    }
    //同样的方式添加家具
    .....
}
```
选好了装修公司，说明以下哪些东西包给装修公司配置，哪些要自己选购（网上买了一个大彩电就不用让装修公司配置了）,然后就动工：
``` java
public static void main(String[] args) {
    //叫来装修公司
    HomeBuilder hbuiler = new HomeBuilder();
    //告诉他配置冰箱和空调，电视我有了，牌子你看着选；
    //这里的牌子货是对应家具的子类（省略了）
    hbuilder = hbuilder.build(new HaierFridge()).build)(new GreeAirCondition());
    //装修公司帮你配置好了你想要的家具，现在网购的电视到了，趁装修公司还没有走，让他帮忙装一下
    hbuilder = hbuilder.build(new TaoBaoTV());
    //现在完成了，让装修公司交付
    Home home = hbulder.build();
}
```
现在完成了，家里的家具齐全了，可以入住了。这里简单举例，现时情况装修要比这复杂的多得多。为什么你只能列举这么简单的例子，毕竟还没有买房嘛...
对于复杂对象，但是对象内容相对固定，而内部的配置又会比较灵活，这样的话建造者是比较合适的。房子即使使精装交付，也可能会缺少一些东西，差一个扫地机器人，投影、电脑等等。这类对象创建出来的时候可能不是那么完善，需要再补充，而且又要灵活补充，像我就用不到电视我就不要build进来。
类似jdk中最常用的StringBuilder，学习设计模式的时候应该找一找优秀框架源码中的相应模式，去体会一下前辈们使怎么想的，我想即使是依样画葫芦，写出来的代码应该也不会差吧。