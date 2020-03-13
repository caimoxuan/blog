---
title: 行为型-职责链模式
date: 2019-11-18 16:02:31
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 有关部门职责链

&emsp;&emsp;有关部门是我国一个神秘的组织，他们没有特定的办公地点、时间、甚至是人员，临时工是这个组织最大的责任承担者。嗯再说就要查水表了...职责链的常用程度绝对是和观察者相当的一个设计模式，而且都是对一个资源的修改的处理模式，不同的是观察者对一个对象的修改需要触发所有的观察动作，而职责链可能在处理的时候存在一定的先后顺序，可能只是其中一个<链>触发相关动作。典型的职责链就好像有关部门一样，当消费者吃了某超时的食物出现中毒症状，家属打电话到消费者协会，消费者协会说这不归我管，你去食监局问问...当消费者到了食监局，他们又说这也不归我管，你去药监局问问...嗯不能再进行下去了，不同的是职责链一般不会走回头路，但是有关部门会，这样就形成了踢皮球来回传。

<!-- more -->
&emsp;&emsp;在许多的地方都能见到职责链的影子，netty中的pipleline，spring中的Filter、Interceptor等等，它其实就是将对象逻辑的处理分类，然后放入一个链表中，遍历链表来处理逻辑；这个逻辑和很多的判断是一样的，类似这样的代码
``` java
public static void main(String[] args) {
    if (args[0] == 1) {
        System.out.print(1);
    } else if (args[0] == 2) {
        System.out.print(2);
    } ...
}
```
这样的代码很常见，处理的方案也有很多，策略模式也是可以的，但是链式模式有一个特性，只要这条链没有被中断，链上所有的节点都能得到处理数据的机会。至于要如何选择，都是要看场景。现在来模拟一个工厂流水线生产水果罐头的流程：
首先我们看一下制作流程： 采摘->清洗->去皮->切块->装罐 大体步骤就这几个吧什么消毒啥的就算了先...
水果要经历这几个步骤，这个水果就是要处理的数据，它有几个属性
``` java
public class Fruit {
    
    /**
     *  编号
     */
    private Integer id;
    /**
     * 苹果 / 梨子...
     */
    private String name;

    /**
     * 处理到哪个步骤
     */
    private HandleStep step;
    /** 
     * 罐装编号
     */
    private Integer code;

    public Fruit(Integer id) {
        this.id = id;
        this.step = HandleStep.PICK;
    }

    //省略get set

    public enum HandleStep {
        //采摘阶段
        PICK,
        //清洗阶段
        WASH,
        //去皮阶段
        SPLIT,
        //切块
        CUT,
        //装罐
        LOAD,
        //完成
        SALE;
    }
}
```
接下来是链上的节点，他们都有个共同的特性，一个处理方法，然后持有下一个节点，可以是一个抽象类
``` java
public Abstract class Handler {

    private Handler nextHandler;

    public void setNextHandler(Handler handler) {
        this.nextHandler = handler;
    }

    abstract Fruit handle(Fruit fruit);

}

//采摘
public class PickHandler extends Handker {

    @Override
    public Fruit handle(Fruit fruit) {
        if (Fruit.HandleStep.PICK == fruit.getStep()) {
            pick(fruit);
        }
        return nextHandler == null ? fruit : nextHandler.handle(fruit);
    }

    private void pick(Fruit fruit) {
        fruit.setHandleStep(Fruit.HandleStep.WASH);
    }
}
//清洗
public class WashHandler extends Handker {

    @Override
    public Fruit handle(Fruit fruit) {
        if (Fruit.HandleStep.WASH == fruit.getStep()) {
            wash(fruit);
        }
        return nextHandler == null ? fruit : nextHandler.handle(fruit);
    }

    private void wash(Fruit fruit) {
        fruit.setHandleStep(Fruit.HandleStep.SPLIT);
    }
}
//切块
public class SplitHandler extends Handker {

    @Override
    public Fruit handle(Fruit fruit) {
        if (Fruit.HandleStep.SPLIT == fruit.getStep()) {
            split(fruit);
        }
        return nextHandler == null ? fruit : nextHandler.handle(fruit);
    }

    private void split(Fruit fruit) {
        fruit.setHandleStep(Fruit.HandleStep.LOAD);
    }
}
//采摘
public class LoadHandler extends Handker {

    /**
     * 这是一个罐子（可能好几个水果才能装一罐呢）
     */
    private Jup jup;

    @Override
    public Fruit handle(Fruit fruit) {
        if (Fruit.HandleStep.PICK == fruit.getStep()) {
            load(fruit);
        }
        if(jup.getFruits().size() > 10) {
            //没有及时换罐子，水果到地上了！
            return null;
        }
        return nextHandler == null ? fruit : nextHandler.handle(fruit);
    }

    private void load(Fruit fruit) {
        fruit.setCode(Jup.getCode())
        fruit.setHandleStep(Fruit.HandleStep.SALE);
    }
}
// 貌似现在多了一个罐子...
public class Jup {
    private Integer code;
    private List<Fruit> fruits;
    public Jup(Integer code) {
        this.code = code;
        this.fruits = new ArrauList(10);
    }
}
```
我们的流水线上的流程齐备了，现在组装流水线开始产罐头：
``` java
public static void main(String[] args) {

    //1号流水线产罐头
    PickHandler ph = new PickHandler();
    WashHandler wh = new WashHandler();
    SplitHandler sh = new SplitHandler();
    LoadHandler lh = new LoadHandler();
    ph.setNextHandler(wh);
    wh.setNextHandler(sh);
    sh.setNextHandler(lh);
    //上述流水线组装的过程可以用建造者改进如果多的话，也可以使用注解，这里就不详述
    //树上的水果，假设已经编好号了
    Jup jup = new Jup(1);
    lg.setJup(jup);
    for(int i = 0; i < 10; i++) {
        Fruit f = new Fruit(i);
        f.setName("apple");
        jup.getFruits().add(ph.handle(f));
    }
    System.out.println(jup);
}
```
这个流水线目前还是很简陋的，手动换罐子，不然就存在浪费水果的风险。这是一个基础的职责链模式的雏形，我们将这些步骤都拆解出来，如果下次我们不需要生产罐头了，我就想吃切片水果，只要改进流水线就可以了，扩展性也十分的好。类似流水线这样，链上每个节点进行加工，处理完成返回是职责链的一种业务模式，当然，也有许多是中途就中断的业务模式，类似web服务的登陆验证，就可以添加一个自定义的filter，发现请求没有通过验证就将请求中断。
这样我们修改业务就十分的方便了，我们可以随意的改变流程，添加流程，调整顺序等等还是那句话，如果业务非常的简单，用设计模式并不会显得优势多大，但是当业务变得复杂，设计模式的优势就体现出来了。