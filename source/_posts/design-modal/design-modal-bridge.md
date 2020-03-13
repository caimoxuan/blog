---
title: 结构型-桥接模式
date: 2019-11-21 16:59:00
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 自由替换桥接模式

&emsp;&emsp;桥接模式特别适用于两个独立变化的维度，什么叫独立变化的维度呢？作为一个吃货，所考虑的东西就是获得的美食，而要获得美食有两个关键的东西，就是厨师和食材。虽说厨师负责处理食材，但是他们的变化实际时独立的。因为好的厨师不论什么种类（前提是优质食材）的食材都能做出美味的食物，也就是食材可以是大闸蟹、熊掌、牛排等等。食材在变，不改变厨师，都可以变成美食。反过来好的食材，换不同的厨师（前提是好的厨师），也能变成美食。几斤小龙虾给星级酒店的厨师烹饪和大排档烹饪一样的好吃（我就爱吃大排档）。

<!-- more -->
&emsp;&emsp;接下来就是一个美食节目，我们准备了20斤小龙虾，20斤牛肉。接下来我们要找星级酒店的大厨和大排档的大厨pk一下，我来做评委。终于知道为什么修仙小说那么受欢迎了，因为自己已经假装自己进入角色了一样...
首先介绍节目的所有东西
``` java
//我们最终的食物
@Data
public class Food {

    // 主要的食材
    Ingredient infredient;
    // 菜名
    String name;
    // 得分
    Integer score;
}

//抽象食材
@Data
public Abstract Ingredient {
    //食材名称
    String name;
    //食材特性
    String desc;

    Ingredient(String name , String desc) {
        this.name = name;
        this.desc = desc;
    }
    
}
// 小龙虾
public class Cray extends Ingredient {

    public Cray() {
        super("小龙虾", "肉少但是紧致，嫩！");
    }

}
// 鲜牛肉
public class Beef extends Ingredient {
    
    public Beef () {
        super("牛肉", "有嚼劲，口感好！");
    }
}


//抽象厨师
public Abstract Cooker {
    //厨师姓名
    string name;

    Cooker(String name) {
        this.name = name;
    }

    //操作cook
    abstract Food cook(Ingredient igd);
}

// 星级厨师
public class StarCooker extends Cooker {

    public StarCooker (String name) {
        super(name);
    }

    public Food cook(Ingredient igd) {
        System.out.print("清蒸" + igd.getName());
        Food f =  new Food();
        f.setIngredient(igd);
        f.setName("蒸" + igd.getName());
        return f;
    }
}

// 大排档厨师
public class StallCooker extends Cooker {

    public StallCooker(String name) {
        super(name);
    }

    publi Food cook(Ingredient igd) {
        System.out.print("十三香" + igd.getName());
        Food f =  new Food();
        f.setIngredient(igd);
        f.setName("十三香" + igd.getName());
        return f;
    }
}
```

好了现在比赛开始，每个师傅有30分钟做出2道菜来：
``` java
public static void main(String[] args) throws Exception {

    //上食材
    Ingredient cray = new Cray();
    Ingredient Beef = new Beef();

    //厨师上
    Cooker startCooker = new StartCooker("jack");
    Cooker stallCooker = new StallCooker("李师傅");

    //30分钟后出菜
    Thread.sleep(30*60*1000);

    Food fsc = startCooker.cook(cray);
    Food fsb = startCooker.cook(beef);
    Food flc = stallCooker.cook(cray);
    Food flb = stallCooker.cook(beef);
}
```

好了现在四道菜摆在我面前，就可以看到这个模式替换非常的方便，今天我吃了星级大厨的菜不好吃了（不太喜欢清蒸）下次我就换中餐厅主厨来试试，今天我吃了小龙虾和牛肉，下次我就换大闸蟹和鲈鱼。但是这就需要对抽象设计需要一定的思考，比如这个例子中的食材的抽象，就不是非常好，因为子类太简单了，会造成类膨胀严重。当然具体使用起来要和实际情况考虑，只是说明一下这个模式的特性，两种变换不相关的东西，衣服和人、图形和颜色、形成一个成品的时候需要两个东西配合。穿衣服的人（牛仔男、草裙女...）,不同颜色的图形(黄圆、紫方...)。遇到这样的场景的时候，可以尝试一下桥接模式。