---
title: 行为型-状态模式
date: 2019-11-20 17:40:01
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 策略表亲状态模式

&emsp;&emsp;大多时候，对于对象的操作避免不了对于对象状态的维护,许多时候我们讨厌状态，但许多时候引入状态确实能解决问题；说到状态模式，很多时候是和策略模式做对比的，因为他们真的很像，并且连类图都是一样的。其实设计模式里面有些设计模式确实可以在一些场景互相替换了之后都比较的优雅，模式不仅仅在模式的结构本身，重要的是如何使用这个模式，往往这个时候区别就体现在使用者的意图上。

<!-- more -->
生活中的事物也是有许多的状态可描述的，比如见天天气就可以用状态来表述：天晴、多云、小雨、大雨、雷雨等表述的是天气的状态；作为一个公民我们自身也是可以有状态表述：普通公名、限制消费的公名、被剥夺政治权利的公名等；事物所处的状态，影响事务的表象，这样的改变是被动的，就像被限制消费的公民，一旦你变成了这个状态，你肯定不需要去办理什么手续，飞机自然就不然你做了。状态模式所强调的是状态的变化影响事务的表象，而策略模式给了你选择状态的余地；
就拿订单状态来几个例子，先看看状态模式的成员：
1. 抽象的状态机制 State
2. 状态机制的实现
3. 上下文Context

状态模式的类图是和策略模式一样的，那么成员自然也是相似的，还是这几个成员，只不过状态的变化对事物状态的表象的影响是同步的，状态变化自然的影响了事物表象；
假设有一个订单对象Order
``` java
@Data
public class Order {

    private Long orderId;

    private OrderStatus status;
    // 创建时间
    private Long timeCreate;
    // 超时关闭时间
    private Long timeExpire;

    // 支付要素 （同一个订单如果没支付可以弹起相同的收银台）
    private String credential;
}
// 订单的几种状态
@Data
public enum OrderStatus {
    //订单初始状态（刚刚完成创建）
    INIT,
    //订单提交未支付
    PROCESS,
    //订单已支付成功
    SUCCESS,
    //订单付款失败
    FAILED,
    //订单已退款
    REFUND,
    //订单关闭
    CLOSED;
}
```
然后是抽象订单状态变化的机制和实现
``` java
public abstract class OrderState {

    Order order;

    OrderState (Order order) {
        this.order = order;
    }

    abstract void doAction();
}

//提交订单
public class ProcessOrderState extends OrderState {

    public ProcessOrderState (Order order) {
        super(order);
    }
    /**
     * 订单从init -> process 用户提交了订单从创建到提交，这时候客户端的动作就是下单，然后提交后台，返回支付确认弹窗
     * 状态校验省略
     */
    @Override
    public void doAction() {
        //提交了订单要的事
        //1 设置订单超时关闭 假设是15分钟 (这里假设有专门的定时任务检测关闭， 或者用消息队列延时消息等)
        order.setTimeExpire(System.currentTimeMillis() + 15*60*1000);
        //2 设置订单的支付要素（就是弹起的收银台）
        // 调用支付sdk或接口获取支付要素，如支付宝网页支付就是一个表单
        String form = "调用支付平台获取的支付要素";
        order.setCredential(form);
        order.setOrderStatus(OrderStatus.PROCESS);
    }

}
//支付成功
public class SuccessOrderState extends OrderState {

    public SuccessOrderState (Order order) {
        super(order);
    }

    @Override
    public void doAction() {
        order.setOrderStatus(OrderStatus.SUCCESS);
        
        System.out.println("发送消息给库存发货");
    }
}
//成功后退款
public class RefundOrderState extends OrderState {

    public RefundOrderState (Order order) {
        super(order);
    }

    @Override
    public void doAction() {
        //校验订单是否能退
        //如果能退查询物流是否发货货物相关处理等
        order.setOrderStatus(OrderStatus.REFUND);
        //这个时候订单已经不能继续操作了，所以也可以将订单关闭
        
    }
}
//订单关闭
public class CloseOrderState extends OrderState {

    public CloseOrderState (Order order) {
        super(order);
    }

    @Override
    public void doAction() {
        //校验订单是否能关闭（成功的订单不能关，还要让人家退款.）
        order.setOrderStatus(OrderStatus.CLOSE);
    }
}
```
最后需要一个上下文来操作订单
``` java
public class OrderContext {

    private OrderState orderState;

    //设置状态机
    public void setState(OrderState state){
        this.orderState = state;
    }

    public OrderState getSatate() {
        return this.orderState;
    }

}
```
现在来模拟用户的几个动作：
``` java
public static void main(String[] args) {
    OrderContext orderContext = new OrderContext();
    //用户下单
    Order order = new Order();
    order.setOrderId(System.currentTimeMillis());
    order.setStatus(OrderStatus.INIT);
    orderContext.setState(new ProcessOrderState(order));
    orderContext.getState().doAction();
    //用户付款
    orderContext.setState(new SuccessOrderState(order));
    orderContext.getState().doAction();
    //用户退款
    orderContext.setState(new RefundOrderState(order));
    orderContext.getState().doAction();
}
```
上面模拟的订单操作和实现都是非常简化的，实际中需要很多的校验，订单都是从数据库中查询，考虑订单状态的幂等等。这里先讨论状态模式，在拿上面的例子做个比方，现在法院要将一个人限制消费，首先会发布一个声明，将这个人加入到被执行人名单中，这个声明就像`orderContext.setState()`，这个人就被标记成了限制消费状态。然后对他执行限制消费`doAction()`，那么带着这个标记，这个人无论去到什么地方消费，都会受到这个状态的影响。再对比一下策略模式，策略将算法进行封装，执行完算法获取到结果之后，除了获取到的结果，其他没有什么东西发生改变了，也就是策略不影响状态。上例子中每次的状态改变都会对应到Order的改变上可以将改变的Order作为参数返回，或者在State中提供方法访问order，一般来说策略模式中的策略只有一个方法，而状态模式的状态会有多个方法，但这也不是绝对的。
总的来说，如果涉及到一个一对多，并且`多`这个改变会影响到`一`的行为，可以使用状态模式，如果不涉及到状态的变化，策略模式模式也能满足，看起来状态好似策略模式的一种特殊版本吧；