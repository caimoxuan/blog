---
title: 结构型-代理模式
date: 2019-11-25 15:22:18
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 隐藏目标代理模式

&emsp;&emsp;代理模式的应用真的是非常的多也是非常的广泛。首先我们从应用的代理开始。nginx作为最常用的反向代理应用，他的原理就是代理模式。nginx所代理的，是之后的多台服务器或者是应用（nginx支持ip和url的代理）。对于用户来说，访问一个使用了nginx的网站，首先请求会到nginx，然后通过nginx配置的负载均衡策略来选一个服务器访问。反向代理就有一个特点，是在服务端配置的，就可以做负载均衡，因为代理的是服务器。而有反向代理，也有正向代理。在国内我们是很难访问国外的网站的，国内兴起了一段时间的vpn（现在都凉的差不多了），vpn和nginx不同，它代理的是用户。没有vpn的时候我们访问`google`实际上也是有一个`vpn`的，这个`vpn`的作用就是不让你访问它...而使用了vpn，这个vpn就代替你去访问你要访问的东西，因为这个vpn可以访问`google`，而用户可以访问这个vpn。所以在使用vpn的时候，可以选择节点，这些节点基本上都是国外的或者是香港台湾等中国一国两制区域。可以发现，vpn的配置是在用户端的。

<!-- more -->

&emsp;&emsp;代理模式在编码中分为静态代理和动态代理。静态代理指的是，代理的对象在编译的时候就已经确定好了，这个被代理的对象已经明确知道是谁了，不能再变化了。动态代理指的是，被代理的对象在编译完之后没有确定，等到程序运行的时候才知道代理的是谁。一般使用的时候，往往都是使用动态代理，很少会有使用静态代理的情况。

先来简单实现一个静态代理,静态代理和装饰基本上就是一模一样，只不过装饰意在加强，代理意在控制。比如现在有个接口，他可能在调用的时候会发生异常，要在异常发生的时候报警。
``` java
// 订单接口
public interface OrderService {
    void submit() throws Exception;
} 

// 下单实现
public class OrderServiceImpl implements OrderService {

    @Override
    public void submit() {
        if (new Random().nextInt(10) > 5) {
            throw new Exception("bad luck!");
        }
    }
}

// 代理
public class OrderServiceProxy implements OrderService {

    private OrderService orderService;

    public void setOrderService(OrderSerivce orderService) {
        this.orderService = orderService;    
    }

    @Override
    public void submit() {
        try {
            orderService.submit();
        } catch (Exception e) {
            //告警 这里建议把异常再次抛出去， 因为包装异常可能会影响客户端的异常处理；
            throw e;
        }
    }
}
```
静态代理就是这样，非常的简单，但是我觉得这不是代理模式的精髓所在。在java中，实现动态代理目前来说有两种方式：
1. jdk的动态代理；
>jdk的动态代理需要被代理的对象必须实现接口，因为是通过反射获取代理对象接口的所有`method`来进行代理，他的核心就是在代理方法中，使用字符串的拼接的方式，创建一个和被代理对象拥有相同的接口方法（通过反射获取method拼接）的类（名称是`$Proxy0`），然后将这个代理类用类加载器加载到内存中使用；他的特点就是，被代理的对象必须实现接口；使用方式这里不详述。这里说一下动态生成的`$Proxy`文件的查看：
>>首先完成jdk动态代理，测试动态代理可以正常运行，然后在测试的main方法中最开始的时候加上`System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true");`,适用于jdk1.8，之后版本替换为`jdk.proxy.ProxyGenerator.saveGeneratedFiles`，然后运行动态代理测试，在classPath目录下会生成一个`com/sun/proxy`文件夹的下面就是这个生成的代理类，可以通过反编译查看。

2. cglib动态代理；
>cglib动态代理可以不用约束被代理的对象是不是实现了一个接口，因为他是通过继承被代理的对象，然后重写其中的方法。当然这个类也是动态生成的。在反射性能还不太好的时候，使用cglib会是一个好的选择，并且可以代理类而不拘束于接口。但是通过继承的方式就能发现，逃不脱java的限制，`private`和`final`的方法并不能重写，所以对于这两种方法是无效的。

如果要问jdk动态代理能代理private方法吗？首先应为jdk动态代理要求被代理类实现一个接口，所以只能代理接口中的方法，然而接口中的方法都是`public`的。但是`final`的方法是可以的。