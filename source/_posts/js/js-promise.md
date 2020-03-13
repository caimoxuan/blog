---
title: Promise是什么，怎么用
date: 2019-12-12 02:02:01
thumbnail: 
categories:
    - js
tags:
    - js
---


## Promise是什么

&emsp;&emsp;首先通过字面来看，他是一个`承诺`，意思就是现在我先答应你，以后一定给你兑现；对应到代码中就是，这里有一个操作，比较费时间，浏览器接受你的操作请求，然后将他包装成一个`承诺`给你，这个`承诺`在之后的摸一个时刻会兑现。在这段时间之间，浏览器可以继续做自己的事情。
&emsp;&emsp;为什么要有这样的操作？我们都知道js是一个单线程语言，所有的代码只能一行一行的执行，那么如果执行了一个耗时的操作，下面的代码只能等待它执行完成然后再执行。这怎么行，node的招牌可是高并发，这怎么并发的起来...所以js一定要异步！
&emsp;&emsp;试想以下当我们读取一个文件的时候，这个时候耗时的操作是系统读取磁盘文件，这个时候浏览器是空闲的，我们将读取文件的操作交给浏览器，浏览器返回给我们一个`承诺`，会把文件读取完成然后告诉我们，让我们继续操作，这就是异步，和java中的nio的epoll模型是一样的；
&emsp;&emsp;那么`承诺`有的时候是会搞砸的，浏览器答应帮你做这件事，但是这件事情能不能成功，那就不是浏览器能控制的了，也就是说事情会搞砸，就是抛出了异常，既然搞砸了，那么浏览器也会告诉你，事情没有办好，但是我把错误信息带来了，你看看要怎么办吧。所以这个`承诺`一定会兑现，但不一定是想要的结果（要有异常处理）;

<!-- more -->

## Promise的三个状态
1. pending [待定状态] 初始化状态，这个时候操作还没有开始处理或没处理完成；
2. fulfilled [实现] 操作成功
3. rejected [否决] 操作失败

Pormise一开始是Pending状态，状态的变化只可能是之后的2种，而且一旦变化了一种状态，状态就保持不变了；


## Promise怎么用

首先最快速的尝试方式，现在打开一个浏览器，F12打开console，在其中输入(不能回车哦回车就是执行了)；
``` js
new Promise(function(resolve, reject){setTimeout(function(){resolve("ok");}, 1000)}).then(function(res){console.log(res)});
```
可以看到1秒后console中打出了`ok`。

我们来看看Promise的几个部分：
`new Promise()`创建了一个承诺；
`function(resolve, reject)`承诺接受一个方法，方法接受2个参数，`resolve`告诉我事情什么情况是办成了，`reject`事情什么时候是搞砸了;
`setTimeout(function(){resolve('ok');}, 1000)`并且告诉浏览器，承诺要做的事情是啥，这里用`setTimeout`来模拟一件耗时的操作，这里只有成功;

这是一个最简单的演示，其中没有处理搞砸的情况，举个例子什么时候会搞砸呢？
当然是在做操作的时候抛出了异常，或者是出现了和预期不符的情况，告诉浏览器搞砸了之后怎么做,我们对上面的代码做修改；

``` js
new Promise(function(resolve, reject){
    setTimeout(function(){
        if(Math.random() > 0.5) {
            resolve("ok");
        } else {
            reject("no");
        }
        }, 1000)})
    .then(function(res){
        console.log(res)
    }).catch(function(err){
        console.log("err:" + err);
    })
```
这个时候这件事情有50%的概率搞砸，一旦搞砸了，`.catch()`中的方法将会执行，那么搞砸这件事可以使用`reject`来搞砸，也可以直接抛异常（这是推荐的方式）`throw new Exception(err)`;

这时一个完成的promise处理流程，一般来说都是这样用的；如果使用es6语法，可以将上述代码简化成：

``` js
new Promise((resolve, reject) => {
    setTimeout(() => {
        if(Math.random() > 0.5) {
            resolve('ok');
        } else {
            reject('no');
        }
    }).then(res => {
        console.log(res);
    }).catch(err => {
        console.log(err)
    });
});
```

## 扩展功能

批量执行

Promise.all([new Promise(...), new Promise(...)]) 批量处理promise接受一个Promise数组作为参数；
特点： 当其中一个失败则全部失败，返回值是第一个失败的Promise的值；当全部成功，返回值是全部Promise的数组；

Promise.race([new Promise(...), new Promise(...)])
特点： 当其中一个执行成功则执行`.then`种的内容；