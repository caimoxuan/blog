---
title: 行为型-命令模式
date: 2019-11-18 10:36:40
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## SDK封装者命令模式

&emsp;&emsp;命令模式，也是一种常见的模式，相信开发对接过一些系统的，基本都用过人家的sdk，比较有特点的就是支付宝开放平台的alipay-sdk，通过这个sdk，我们能够快速的对接支付宝开放平台的接口，自己不用关心验签加签返回值解析封装等操作。像这样的sdk封装，就非常具有代表性，并且他的特性也很鲜明，易用性使用者能够切实的体会的到，并且对于sdk的开发者来说，扩展性也很好。

<!-- more -->

&emsp;&emsp;首先了解一下典型的命令模式的成员：
1. 命令的接口 ICommand 接口类；
2. 命令的实现 Command 类；
3. Client 客户端；
4. Invoker 调用命令的类；
5. Receiver 接收并执行的类；

来看看这些成员各自实现的功能，最后结合alipay-sdk来分析一下：
<1> 和 <2> 是比较好理解的就是一个接口(或抽象类)，一个实现；
``` java
public interface ICommand {
    public void execute();
}

public class Command {

    @Override
    public void execute() {
        System.out.println("do something");
    }
}
```
<3> 是一个客户端，他真的就是客户端，都可以理解为测试main程序，就类似sdk的调用者。这里面就是将这些成员创建后调用；

<4> 负责直接调用命令，这样的好处是可以通过一个Invoker来调用不同的命令，将命令的实现和调用分离。个人觉得这是一个附加项，当命令多的时候优势才会体现出来。
``` java
public class Invoker {
    
    private ICommand command;

    public Invoker(Icommand command) {
        this.command = command;
    }

    public void action() {
        this.command.execute();
    }

}
```
<5> Receiver是实际的业务处理者，他负责接授命令并处理业务
``` java 
public class Receiver {
    public void receive() {
        System.out.println("业务处理");
    }
}

//然后在Command中可以选择持有Receiver
public class Command {

    private Receiver reveiver;

    Command(Receiver reveiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        this.reveiver.receive();
    }
}
```
最后就是通过 <3> 这个成员来调用Invoker，但是这样的脱离实际场景的东西还是很抽象，然后让我们看看alipay-sdk中是怎么做的：
1. 首先AlipayClient就相当于ICommand： 

![command-1](alipayclient.png)

类图所示，一般都是用DefaultAlipayClient， 嗯目前就只有这个...

2. 进入AlipayClient 

![command-2](alipayclient-class.png)

发现它声明了许多的命令，这些命令都有个特点，以AlipayRequest为参数，以AlipayResponse为返回值。

3. 查看AlipayClient中excute的实现：

其中的实现就是处理加签、解析请求数据、发送http请求、返回数据组装、返回AlipayResponse。


其中代码的逻辑就不细细说明，主要专注于成员的说明，这其中alipay-sdk给我们提供了
1. 命令接口 （AlipayClient/AbstractAlipayClient）
2. 命令实现 （DefaultAlipayClient）
3. Receiver (这里其实被弱化成了一个参数： AlipayRequest 并且业务不是由它处理)
4. client 谁使用到了alipay-sdk 他就是一个客户端；

可以发现少了一个Invoker的成员。嗯这个成员可以有，也可以不存在，如果逻辑就是调用一个支付宝开放平台接口，搞一个类实例化一个DefaultAlipayClient调用就完成了。但是如果说下次来了一个微信的平台对接需求，银联的对接需求，虽然他们没有sdk... 但是可以自己封装一个sdk（照着alipay-sdk就行），然后添加一个Invoker做统一命令提交（可能有的时候还是不必要，设计模式就是这样，不应该生搬硬套，结合场景合理的才是最好的）。

>可以总结一下，命令说白了就是对请求的封装，他的好处方便了客户端，同时对开发来说又有良好的扩展性。
>对于客户端来说，对接拥有sdk的平台是比较轻松的，因为它包装了加签验签的逻辑、请求调用的逻辑、参数组装的逻辑等等；试想一下，如果对接的是一个平台，这些做了也就没什么。但是对接许多的平台的话，每家平台有不同的验签规则，什么奇怪的都有，并且有的平台还不知道如何验证签名没问题。然后自己发起请求，请求的格式又是千奇百怪。post请求、get请求、urlencode的、form表单的、放在post的body中的...然后是组装数据，xml格式数据、json格式数据、对照着人家的文档一个字段一个字段的封装成类...完成了这一系列的操作，最后再预估一下人家接口平均调用时间，设置了http调用超时时间完成调用。然后报了一个验签失败！半天找不到为什么！事件都浪费在了这种操作上。这都是因为平台没有提供sdk！
>对于sdk的开发者，提供平台sdk是一个辛苦的活这不假，但是自己做的平台怎么用自己是最清楚的。而且，sdk只要开发者开发一次，方便的可是成千上万的对接者。