---
title: 行为型-模板方法模式
date: 2019-10-22 23:11:17
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 拿来可用的模板方法

&emsp;&emsp;有开发过企业级java应用的肯定不会对这个模式陌生，也许正在进行着的CURD都包含着模板方法的使用。
&emsp;&emsp;简单的mvc模式下，dao层负责数据层的CURD，话说CURD这几个方法够模板了吧，哪个数据操作层能脱离CURD，CURD被誉为最底层的工作，但是复杂的CURD也可以非常复杂，特别是在数据爆炸的今天，操作的会是几个数据源。话说回来，dao中的UserDao、OrderDao这样的类是不是都有CURD方法，那么这也太模板了吧，是不是可以来一个BaseDao， 将CURD放入BaseDao中，其他的dao继承BaseDao，就都有了CURD方法，而各个dao中特殊的方法，也可以在各自的dao中实现。这是模板方法的一种模式，还有一种类似于kafkaTemplate、mongodbTemplate、redisTemplate等等，这些开发过spring应用的应该不会陌生。当要使用一些中间件的时候，这些中间件的操作都是固定的形式（CURD和其扩展），直接将人家定义好的模板方法拿来使用就可以完成对这些中间件的操作。
&emsp;&emsp;上面的2种方式在开发种都比较常见，还有一种不常见的，但是也许有过类似的经历：

<!-- more -->

简单来说一个例子：公司今天新来了一个实习生，刚来就被分配到了一个任务，给定一个整数数组，按从大到小排序然后输出。接到任务的时候，知道以自己4年大学的经验，完全能够应付将一个数组输出的任务，但是还要排序，这可咋办，义务教育没有教啊。但是架构师已经把架子给你搭好了:
``` java

public abstract class AbstractArraySort {
    
    //排序的部分（难度较大一点需要借助别人来完成）
    protected abstract void sort(int[] arr);
    
    //需要实习生实现的部分
    public void printSortedArray(int[] arr) {
        
    }
}
```
然后实习生通过义务教育得到的知识三下五除二：
``` java
public void printSortedArray(int[] arr) {
    //用上面的方法排序
    this.sort(arr);
    //输出
    for (int i =0; i < arr.length; i++) {
        System.out.print(arr[i] + " ");
    }
}
```
好了这个时候你求助了你的师兄，把上面的类交给了他，师兄一看，顺手把他实现了：
``` java
public class ArraySortTemplate extends AbstractArraySort {

    @Override
    protected void sort(int[] arr) {
        Arrays.sort(arr);
    }

} 
```
师兄写完之后，顺带夸了自己一把，说自己使用的双轴快排，好让实习生仰视一番。实习生拿过来一用：
``` java
public static void main(String[] args) {
    int[] arr = {3,5,6,7,1,2};
    AbstractArraySort aas = new ArraySortTemplate();
    aas.printSortedArray(arr);
}
```
发现没有问题，觉得大佬果然牛逼。

&emsp;&emsp;像上面这样，由一个抽象类和一个或多个实现类组成的，抽象类中一般有三类方法：
>1. 模板方法（本例中的`printSortedArray`），一般会调用抽象方法来完成逻辑，可声明为`final`方式子类重写逻辑；
>2. 抽象方法（本例中的`sort`），需要子类去完成的逻辑，抽象类定义方法的规范；
>3. 钩子方法（本例中未体现），抽象类中声明并实现的方法，子类可以通过调用它来改变模板方法的逻辑，改造上面的例子。现在假设我们需要控制排序是从小到大还是从大到小：

先在抽象类中添加钩子方法返回排序类型，并在模板方法中加以控制
``` java 
//AbstractArraySort 添加的钩子方法 默认从小到大
public boolean hook() {
    //true false 分别对应从小到大和从大到小；
    return true;
}

//添加一个从大到小的排序

//修改模板方法
public void printSortedArray(int[] arr) {
    //排序
    this.sort(arr);
    
    //添加hook的影响
    if(this.hook()){
        //顺序输出
        for (int i =0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    } else {
        //倒序输出
        for (int i = arr.length - 1; i >= 0; i--) {
            System.out.print(arr[i] + " ");
        }
    }
} 
```

然后子类可以对钩子函数加以改变：
``` java
public class ArraySortTemplate extends AbstractArraySort {

    //加一个变量使之灵活一点
    private boolean hookType;

    public ArraySortTemplate() {
        //构造函数不声明hookType默认true
        this.hookType = true;
    }

    public ArraySortTemplate(boolean hookType) {
        //人为控制顺序
        this.hookType = hookType;
    }

    @Override
    protected void sort(int[] arr) {
        Arrays.sort(arr);
    }

    //重写 钩子函数 返回变量控制
    @Override
    public boolean hook(){
        return this.hookType;
    }

} 
```

现在可以通过改变钩子方法来改变模板的执行：
``` java
public static void main(String[] args) {
    int[] arr = {3,5,6,7,1,2};
    //手动控制钩子
    AbstractArraySort aas = new ArraySortTemplate(false);
    aas.printSortedArray(arr);
}
```
钩子方法就是这样的一个作用， 这里的例子也许比较粗糙，基本上能描述个大概了。一开始了解这个钩子方法的时候，由于先学过一些前端技术，前端称之为钩子方法的是处理一个回调，因为js能将函数作为方法的参数，而且很多地方没有详细的介绍钩子方法如何使用的，希望可以通过这个例子，使之印象加深吧。