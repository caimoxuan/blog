---
title: 结构型-装饰模式
date: 2019-11-22 11:25:57
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 功能增强装饰器

&emsp;&emsp;在使用的一些框架的时候，常常会见到一些以`Wapper`结尾或者是`Decoator`结尾的类，在使用设计模式的时候，最好能在使用了设计模式的类中加上关键字。由于使用了设计模式，原先代码的逻辑不是那么简单易懂，所以加上标识来让别人更好的理解。对于装饰器来说，一般看见了Wapper就应该是用了装饰器模式。

<!-- more -->
&emsp;&emsp;射击游戏一直都被许多人喜爱，《绝地求生》这款游戏在2018年可谓是最火爆的游戏了。而游戏中最关键的道具就是枪械和配件。即使是没有玩过游戏的同学，也应该知道狙击枪是一种具有强大单体杀伤力的远距离武器，那么问题来了，手上有一杆AWN，是不是会去搜一个倍镜呢？哪怕是4倍...再不行2倍啊...除了倍镜，那么在狙击枪的身上需要用到多少配件呢？在这里，狙击枪就是一个现有的物件，但是通过一些配件的装饰，就可以将原先的狙击枪变成大杀器，接下来就模拟这样一个场景。

首先我们捡到了一把AWN，AWN也是属于枪的一种。
``` java
@Data
public abstract class Gun {

    // 伤害
    private Integer harm;
    // 射程
    private Integer range;
    // 后坐力
    private Integer recoilForce;
    // 声音大小
    private Integer voice;
    // 使用枪的视线距离， 远了就看不见，命中率就低
    private Integer signt;
    // 子弹数量
    private Integer bullet;

    Gun(Integer harm, Integer range, Integer rf, Integer voice) {
        this.harm = harm;
        this.range = range;
        this.recoilForce = rf;
        this.voice = voice;
        this.signt = 200;
    }
    // 瞄准 返回清晰度 影响命中率 这里的目标就不作为类了
    abstract Integer aim(Integer distance);

    // 射击多少距离的目标
     abstract Integer shoot(Integer distance);

}

public class Awn extends Gun {

    

    public Awn() {
        //统一的参数
        super(100, 1500, 20, 180);
    }

    @Override
    public Integer aim(Integer distance) {
        return Math.ceil(this.getSignt() / distance * 100); 
    }

    @Override
    public Integer shoot(Integer distance) {
        //返回这个距离的伤害值 有效射程1500米， 0-1500每多15米伤害减少1；
        if (distance >= this.getRange()) {
            return 0;
        }
        return this.getHarm() - Math.floor(distance / 15);
    }
}
```
现在狙击枪是捡到了，但是没有装上倍镜，这个时候我们试一试这个时候我们打开狙击镜：
``` java
public static void main(String[] args) {
    Gun awn = new Awn();
    awn.aim(800);
    //这个时候发现看不清！我们在看不清目标的情况下，命中率可是大打折扣！
}
```
这个时候对于狙击手来说，这个效果显然是不理想的，那么我们需要给枪安装上倍镜，安装完成倍镜的枪还是原来的枪，只不过多了倍镜的`装饰`；
``` java
// 倍镜也属于配件的范畴
public Abstract class Part extends Gun {

    private Gun gun;

    Part(Gun gun) {
        this.gun = gun;
    }

    // 倍镜并不改变射击伤害
    @Override
    public Integer shoot(Integer distance) {
        return gun.shoot(distance);
    }

    // 其他的配件不改变瞄准精度
    @Override
    public Integer aim(Integer distance) {
        return gun.aim(distance);    
    }

}

// 这是一个4倍镜
public class Telescope4XGunWapper extends Part {

    public Telescope4X(Gun gun) {
        super(gun);
    }

    // 对原有的精度进行加强
    @Override
    public Integer aim(Integer distance) {
        return gun.aim(distance) * 4;
    }
}
```
现在来安装四倍镜使用：
``` java
public static void main(String[] args) {
    Gun awn = new Awn();
    Gun awn4x = new Telescope4XGunWapper(gun);
    // 这时候的精度就提升了4倍 爆头就轻松多了
    awn4x.aim(800);
    // 当然，我们现在就装了倍镜， 还可以装 消音、托腮板、子弹匣等，我们可以在修饰的基础上继续修饰
    Gun awn4xSlience = new SliencePart(aim4x);
    Gun awn4xSlienceMoreBullet = new BulletPart(awn4xSlience);
    ...
    // 经过几次的装饰，就是传说中的满配AWN...
}
```
对于装饰器模式来说一般实现的方式是继承， 这样的方式耦合度是比较高的，也可以使用接口的方式，具体的可以看场景来使用。在设计模式中，装饰模式和代理模式中的静态代理是很相似的，可以说除了使用意图，完全一样。
装饰模式的意图是增强原有的功能，并不影响原先功能的使用，在影响功能的同时，`需要兼顾到其他的功能`；
代理模式可以完全的控制原有的功能，并且可以随意的变化，好像这个意思： `我只要用到了原有的功能就可以了`
当然在一些简单的使用上来说（比如前后打日志），代理和装饰都可以使用，但是代理模式的重点，可不是静态代理，动态代理才是代理的精髓所在。