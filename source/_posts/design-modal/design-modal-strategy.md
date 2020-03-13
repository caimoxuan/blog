---
title: 行为型-策略模式
date: 2019-11-18 17:46:09
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 做个选择策略模式

&emsp;&emsp;策略模式就是选择，选择自己需要的对象来处理数据，它是最常用来简化if else嵌套的一个模式，通过策略模式，我们能最快的简化复杂的条件判断的嵌套，而每一个策略所关注的是对一种数据的处理方式，最常举的例子就是计算器中的加减乘除的策略模式的实现；策略模式也常和另一种模式：模板方法做比较，通过比较发现，策略模式和模板方法都对数据处理的方法进行了封装，但是不同的是策略模式有一个选择的过程，而这是模板方法模式中没有的。

<!-- more -->

&emsp;&emsp;先来看看主流的策略模式是怎么样的,还是以最简单的加减乘除举例：
1. 首先是策略接口
``` java
public insterface IStrategy {
    int execute(int x, int y);
}
```
2. 其次实现的策略
``` java
public class AddStrategy implements IStrategy {
    @Override
    public int execute(int x , int y) {
        return x + y;
    }    
}
//其他的就涉略步骤
public class SubStrategy implements Istrategy {...}
public class MutStrategy implements Istrategy {...}
public class DivStrategy implements Istrategy {...}
```
3. 最后给一个选择的余地（就是一个计算器，由使用者选按钮）
``` java
public class Calculator {
    private IStrategy strategy;

    public setStrategy(IStrategy strategy) {
        this.strategy = strategy;
    }

    public int calculate(int x , int y) {
        strategy.execute(x, y);
    }
}
```
最简单的策略就完成了，但是在使用计算器的时候我们还是感觉到比较麻烦的:
``` java
public static void main(Stirng[] args) {
    Calculator cal = new Calculator();
    IStrategy add = new AddStrategy();
    int result = cal.calculate(add);
}
```
可以看到，这样的调用方式貌似是不太合理的，每次我们使用加运算符，却要重新创建一个加的按钮，那么我们可以优化一下；计算器里面的按钮都是计算器的元素，初始化的时候应该就是在里面的。
``` java
public class Calculator {

    public Map<String, IStrategy> buttons;

    public Calculator() {
        this.buttons = new HashMao<>();
    }

    public void init() {
        buttons.put("-", new AddStrategy());
        buttons.put("+", new SubStrategy());
        buttons.put("*", new MutStrategy());
        buttons.put("/", new DivStrategy());
    }

    public int calculator(int x, int y, String code) {
        return buttons.get(code) == null ? 0 : buttons.get(code).execute(x, y);  
    }
}
```
这下我们可以和和使用计算器一样来使用它了，不用每次都创建一个按钮出来；但是这样还是有一丢丢的风险，比如我想传一个<^>符号进行一个幂计算，但是运算当前是不支持的，其实想要约束输入很简单，只要将入参变成一个枚举就可以了。结合map我们可以更加灵活的使用策略模式，相信也有见过另一种使用枚举实现的策略模式。
最早接触枚举的时候对它的印象就是列举出一些状态，没想到这个东西里面居然也能写逻辑：
``` java
public enum Calculator {

    ADD {
        @Override
        public int execute(int x , int y) {
            return x + y;
        }
    },
    SUB {
        @Override
        public int execute(int x , int y) {
            return x - y;
        }
    },
    .....

    abstract int execute(int x, int y);
}
```
枚举策略十分的方便简洁，但我个人认为类似这种简单的算法可以使用枚举策略，但是如果策略比较复杂，比如需要其他类的聚合等，最好不要破坏类的功能比较好；

