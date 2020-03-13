---
title: 浏览器是如何执行js
date: 2019-12-13 22:22:31
thumbnail: 
categories:
    - js
tags:
    - js
---

## 单线程的js如何在浏览器中执行

js是单线程的，众所周知，那么他是如何实现操作页面、响应请求、定时任务、数据操作...？类似在java中可以使用Timer或者是ScheduledThreadExecutor来开启线程执行定时任务等；于是看了一下js在浏览器中如何执行的发现，原来js是单线程，但是浏览器是多线程啊...

<!-- more -->
首先浏览器的工作线程（相对于一个窗口）:
1. javascript 执行线程 （js引擎）
2. 事件触发线程 （EventLoop轮询线程）
3. 界面UI渲染线程
4. http请求线程 （ajax请求会让浏览器新建一个http请求线程来执行）

执行线程：js在运行的时候，js引擎提供了一个消息队列，可以认为是一个任务队列，新的任务在尾端，排队执行；如果有比较耗时的任务，类似sleep，那么后面的任务就得不到执行，页面就会陷入假死状态，这是编程的时候需要注意的；

事件触发线程：当执行线程遇到类似io操作这样非阻塞的任务的时候，会将任务给EventLoop线程去操作，然后执行线程继续向下运行，等到io操作完成，EventLoop线程就会将完成的消息放入执行线程的消息队列中（必须有回调函数，否则会被忽略），执行线程取出消息队列中的事件使用事先设置好的回调函数完成操作；

具体步骤：

到这里先分析一下执行线程和事件触发线程的配合：
在js中存在2种任务：
1. 同步任务，类似sleep()、Math计算等，同步执行的任务，这样的任务会进入执行栈（不如说是执行队列），任务会按FIFO顺序执行；
2. 异步任务，类似io操作，setTimeout中的任务。这样的任务会进入消息队列，并且需要指定回调函数，这样的任务在执行完成之后，注意此时如果执行线程的执行栈是空的，那么异步任务才会进入执行线程的执行栈中执行；执行线程在执行完成执行栈中的同步任务后，会去消息队列中看看有哪些事件可以触发（这一步是线程栈空闲的时候不断进行的），将可以触发（类似setTimeout如果时间没有到，就不会触发）的任务放入执行栈中执行；

然后继续分析线程

界面UI渲染线程：既然要渲染页面，就是将dom的样式属性渲染可视化，但是，执行线程是会修改dom的，你一边改我一边渲染那怎么行，所以执行线程和UI渲染线程是互斥的，所以执行线程会阻塞UI的渲染；

http请求线程：执行http请求的线程，但是不同的是ajax的执行是基于new XMLHttpRequest()来实现的，所以每次都会开一个新的http请求线程来完成；

最后我们看看setTimeout是如何执行的呢？

 setTimeout(function() {console.log("fire");}, 1000); 再执行的时候，执行线程直接将它放入消息队列（因为它已经是一个可触发的事件了，接下来就是看时机，不像io还需要EventLoop轮询是否完成），当执行线程线程栈空闲的时候，会将消息队列中的setTimeout取出来看看是不是能执行了，能执行就调用他的回调函数（这里指setTimeout的第一个参数），完成执行；

结合几个setTimout的案例来体会一下：
1. 首先从最简单的开始,检验setTimeout的执行的顺序；
``` js
console.log("test start");

setTimeout(function(){console.log("timeout");}, 0);

console.log("test end");

```
最后执行的结果是最后打印的 timeout ， 我们执行了一个0延时的任务，这至少说明了setTimeout和正常的执行语句是不一样的，虽然不能证明它去了消息队列，但是至少开始的时候它不在线程栈中，在另一个地方排队存在着；

那么再来验证一下：`线程栈空闲的时候才会将消息队列取出消息执行`；
``` js
console.log("test start");

setTimeout(function(){console.log("timeout");}, 0);

var date = new Date();
while(true) {
    var d = new Date();
    if(date.getTime() + 1000 < d.getTime()) {
        console.log("trigger");
        break;
    }
}

console.log("test end");

```
上面我们模拟一个类似java中`sleep`的操作，将线程阻塞1s，然后我们的setTimout还是无延时触发的，但是最后打印的还是 timeout, 可以说明执行线程在非空闲的时候才会去做其他的事（这里指去消息队列获取消息并执行，但是这点还没有证明是不是这样）。

那么如何证明消息队列中的顺序是FIFO的呢？这里已经假设setTimeout的处理就是去了消息队列；
``` js
console.log("test start");

setTimeout(function(){console.log("timeout1");}, 100);
setTimeout(function(){console.log("timeout2");}, 0);
setTimeout(function(){console.log("timeout3");}, 0);
setTimeout(function(){console.log("timeout4");}, 0);

console.log("test end");
```

可以看见执行结果，2 3 4 的输出顺序可以说明是FIFO，1设置了100毫秒的延时，说明执行与否是需要判断时间；

关于setTimout的执行时机，还有一个经常会犯的错误：
``` js
for(var i = 0; i < 10; i++) {
    setTimeout(function(){console.log(i)}, 0);
}
```
这个的执行结果是打印的全是 10 ；因为setTimout是放入了消息队列的，等待循环进行完毕（循环是执行线程执行），然后再开始处理setTimeout，这个时候i的值已经是10了，要解决这个问题，可以使用js的闭包, 或者使用 let 关键字：

``` js
for(var i = 0; i < 10; i++) {
    function test(val) {
        setTimeout(function(){console.log(val);}, 0);
    }
    test(i);
}

for(let i = 0; i < 10; i++) {
    setTimeout(function(){console.log(i);}, 0);
}
```