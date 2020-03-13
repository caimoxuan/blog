---
title: 结构型-适配器模式
date: 2019-11-21 14:32:13
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---


## 能用就行适配器

&emsp;&emsp;适配器模式一般来说应用在一个运行良好的系统，由于一些需求的出现导致需要修改一些系统功能来使得对新功能的兼容。生活中应该都遇到过这样的事情，家用电器的插头有三孔的插头，有两孔的插头，三孔的插头比两口的插头多了一个地线脚，由于一些电器的功率比较大，或者是有金属外壳亦或是有潮湿漏电的可能。这样的电器如果用两孔插头，金属外壳可能就会带电，有安全隐患。如果说想要让电器正常工作的话，两孔的插头已经足够了。

<!-- more -->


&emsp;&emsp;有的笔记本是使用金属外壳的，在设计的时候就是三孔的插头，像苹果笔记本外壳就是金属的，但是他的插头是两孔的，我就体会过以便打字一边被电的感觉...应为他本身的功率不大，机身的电流就不会很大，所以电到了也就是一点麻麻的感觉。而两孔插头的确在大多地方比较常用，所以三孔插头可能会准备一个两孔插座的适配器。界限来就模拟一下这个场景。

从插头开始：市面上有两孔和三孔的；
``` java
public interface Plug {
    //获取插脚数
    Integer getLine();
}
//三孔
@Data
public class TriplePlug implements Plug {

    @Override
    public Integer getLine() {
        return 3;
    }
}

@Data
public class DoublePlug implements Plug {

    @Override
    public Integer getLine() {
        return 2;
    }
}
```


在没有适配器的时候三孔的插头需要三孔的插座，两孔的用两孔的插座：
``` java
public interface Base {
    void use(Plug plug);
}
//三孔
public class TripleBase implements Base {

    @Override
    public void user(Plug plug) {
        if (Integer.valueOf(3).equals(plug.getLine())) {
            System.out.println("接入成功");
        } else {
            System.out.println("接入失败");
        }
    }
}
//两孔
public class DoubleBase implements Base {

    @Override
    public void user(Plug plug) {
        if (Integer.valueOf(2).equals(plug.getLine())) {
            System.out.println("接入成功");
        } else {
            System.out.println("接入失败");
        }
    }
}
```
可以发现现在三孔的插头需要三孔的插座才能成功接入，现在我们买了一个三孔->两孔的插头适配器
``` java
public class TripleToDoubleAdapter implements Plug {

    private TriplePlug plug;

    public TripleToDoubleAdapter(TriplePlug plug) {
        this.plug = plug;
    }

    @Override
    public Integer getLine(){
        System.out.rintln("adapter change plug line:" + plug.getLine() + " to 2");
        return 2;
    }
}
```
接下来我们来我们的三插头笔记本就能正常工作了：
``` java
public static void main(String[] args) {
    //三孔插头
    TriplePlug computerPlug = new TriplePlug();
    //两孔插座
    Base doubleBase = new DoubleBase();
    //试试看先
    doubleBase.use(computerPlug); //明显插不进去啊 倒是见过吧地线直接掰弯，强行插入的...

    //买来适配器 将电脑插头放入适配器
    Plug computerPlugAdapter = new TripleToDoubleAdapter(computerPlug);
    //使用适配器
    doubleBase.use(computerPlugAdapter);
}
```
对于上述的适配，也可以将两孔的插头适配到三孔的插座中，但是过多的适配也是有问题的，试想适配器的类型也是插头类型，类可不像真正的插头那样直观，在适配器多的时候很容易就搞错了，所以对于过多的适配，最好的方式还是重构系统，以防不好把握代码的整体。想想为什么不把插头都统一成两孔呢？适配三孔插头为两孔，那肯定还是有安全隐患的。
