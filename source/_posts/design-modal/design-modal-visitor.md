---
title: 行为型-访问者模式
date: 2019-11-12 16:43:10
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 统一规划访问者

&emsp;&emsp;按照书中所述（具体忘了是那里看的），这个模式可能是行为型模式中最难的一个模式了。个人觉得的确是这样，第一次看这个模式的时候，完全是一脸懵逼，完全想不出可以利用的场景。觉得可能用的比较少吧，就这样没有完全的理解它的意图。个人认为理解一个设计模式的第一步，就是能自己找到使用的场景，并且比较合理不会生搬硬套的。第二部就是能对这个模式进行一些自己的改进和适配。最后异步就是真的融会贯通，能根据不同的场景，结合不同设计模式不同的变种，可能写完才发现自己原来用了设计模式。

<!-- more -->

## 初识访问者
&emsp;&emsp;先了解这个模式解决的问题：将数据的结构和数据的操作分离；为什么要这样做，如果数据的结构可能是易变的，数据的操作也是易变的，如果操作和数据不分离，那么对原来的改动可能会比较大；好吧，有的东西就是不容易说，先看一个我觉得比较好的例子,访问这模式到底在干嘛：
``` java
//被访问者
public class Element {

    public void doVisit() {
        System.out.println("element do visit!");
    }

    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

}
//访问者
public class Visitor {
    public void visit(Element e) {
        e.doVisit();
    }
}
//测试
public static void main(String[] args) {
    new Element().accept(new Visitor());
}
```
一个访问者的雏形就呈现出来了，可以看见，被访问者接受访问者的对象来访问自己（accept(Visitor)），而访问者可以利用访问获取到的被访问对象，从而调用到被访问者对象中的方法；因为这是一个雏形，也许体会不到这种模式的好处，但是基本能知道他干了什么。

可以看到雏形中的代码的耦合性是非常强的，毕竟这不是观察者模式的全部，接下来对他进行改进,但是一般的模板式的代码个人觉得不是非常的有感觉，那么先来设计一个场景。现在有一个需求，假设我有一个管控系统，里面有支付，退款这2种报表要输出，而且对报表也有需求，就是需要一个表头在上的报表，和一个表头在左的报表（先不考虑需求合不合理吧）。
第一步将被访问者接口化
``` java
public interface IElement {
    void accept(IVisitor visitor);
}

public class Charge implements IElement {
    
    private String chargeId;

    private String productName;

    private Long timeStamp;
    ...

    //省略get/set
    
    public Map<String, Object> getData() {
        Map<String, Object> map = new HashMap();
        //这里可以用反射获取键值对或者用JSONObject，但是为了体现charge 和 refund的不同，因为相同的话直接可以写在抽象类里
        map.put("chargeId", this.chargeId);
        ...
        return map;
    }

    @Override
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }

}

public class Refund implements IElement {

    private String refundId;

    private String chargeId;

    ....

    public Map<String, Object> getData() {
        Map<String, Object> map = new HashMap<>();
        map.put("refundId", this.refundId);
        ...
        return map;
    }

    @Override
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
}
```
第二步将访问者接口化
``` java
public interface IVisitor {
    void visit(IElement element);
}

public class VisitorTop implements IVisitor {

    Map<String, List<Object>> datas;

    public VisitorTop() {
        this.datas = new HashMap<>();
    }

    @Override
    public void visit(IElement element) {
        Map<String, Object> map = element.getData();
        //填充数据
        map.forEach((k,v) -> {
            List<Object> row = datas.get(k);
            if(row == null) {
                row = new ArrayList<>();
                row.add(v);
                datas.put(k, row);
            } else {
                row.add(v);
            }
        });
        
    }

    public void printTopTable() {
        //表头在上的报表打印
    }

}

public class VisitorLeft implements IVisitor {

    Map<String, List<Object>> datas;

    public VisitorLeft() {
        this.datas = new HashMap<>();
    }

    @Override
    public void visit(IElement element) {
        //这里其实和VisitorTop相同，这里应该有不同的处理逻辑，本例中相同，不同体现在报表的打印上；
        Map<String, Object> map = element.getData();
        //填充数据
        map.forEach((k,v) -> {
            List<Object> row = datas.get(k);
            if(row == null) {
                row = new ArrayList<>();
                row.add(v);
                datas.put(k, row);
            } else {
                row.add(v);
            }
        });
    }

    public void printLeftTable() {
        //打印表头在左的报表；
    }
}
```
这样我们就形成了一个报表的打印组合，看上去已经比较灵活了：
>支付订单 -- 顶部表头报表
>退款订单 -- 顶部表头报表
>支付订单 -- 左部表头报表
>退款订单 -- 左部表头报表

已经可以实现上面的功能了，但是这里还缺多少一个重要的步骤，报表有很多条数据的呀，这个数据谁来提供。这里就需要访问者中另一个重要的角色，它负责获取批量的被观察对象,这里一个简单的伪代码；
``` java 
public class TableManager {

    public List<Refund> listRefund() {
        //从数据库中查出退款数据
    }

    public List<Charge> listCharge() {
        //从数据库中查出订单数据
    }
}
//然后开始测试
public static void main(String[] args) {
    //获取数据
    TableManager manager = new TableManager();
    List<Refund> refunds = manager.listRefund();
    List<Charge> charges = manager.listCharge();
    //创建观察者
    IVisitor topVisitorForRefund = new VisitorTop();
    Ivisitor leftVisitorForRefund = new VIsitorLeft();
    IVisitor topVisitorForCharge = new VisitorTop();
    Ivisitor leftVisitorForCharge = new VIsitorLeft();
    //开始组装报表
    refunds.forEach(r -> r.accept(topVisitorForRefund));
    refunds.forEach(r -> r.accept(leftVisitorForRefund));

    charge.forEach(c -> c.accept(topVisitorForCharge));
    charge.forEach(c -> c.accept(leftVisitorForCharge));
    //分别打印4张报表 这里统一成接口也可以但是为了区别开来；
    topVisitorForRefund.printTopTable();
    leftVisitorForRefund.printLeftTable();

    topVisitorForCharge.printTopTable();
    leftVisitorForCharge.printLeftTable();
}
```
上面就完成了各种报表的需求，试想以下按常规方案完成这个需求需要怎么做呢。
首先我们一样查出数据的list；
然后将专门有个类来组装数据，有多少种数据就需要创建多少种组装数据的方法；
然后在需要一个类输出数据（可能实际需求是输出成excel格式然后命名成不同的名称导出之类的），这也需要一种数据一个方法；
最后依次调用这些方法完成；
等等...好像听起来复杂度和原来差不了多少啊...
是的，设计模式在代码并不复杂的情况下收效甚微，这也就是为什么说不要滥用设计模式的原因，这其中分寸的把握需要长时间的历练和参透，目前对自己的希望就是写的代码能让自己舒服一点，没有什么大道理。
再自己体会下访问者模式的特点，将数据结构和数据的操作分离，数据的结构是被访问者， 数据的操作是访问者，数据结构的修改不会影响到数据的操作，数据操作的修改也不会影响到数据的结构，这样的结构最后用一个访问的动作连接起来，它的扩展性就非常的好。自由的添加访问者类，也可以自由的添加被访问的对象。毕竟在一个报表系统中，需要的可能是各种各样的报表，也需要各种各样的数据来生成报表。