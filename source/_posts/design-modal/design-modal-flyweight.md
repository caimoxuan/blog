---
title: 结构型-享元模式
date: 2019-11-25 10:20:26
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 重复利用享元模式

&emsp;&emsp;我们在开发系统的时候，现在一般来说基本都会用到redis或者是memcached这样的缓存系统，对于分布式系统来说，如果使用程序本地缓存，那么每个服务都需要在本地维护一个HashMap这样的缓存，如果有10个服务就会维护10个本都缓存，先不说这些的缓存一致性或者是数据丢失的问题。首先一个系统是否运行稳定一个很关键的因素是对内存的使用是否合理，像这样维护大量对象在内存中的情况，一个是可能数据重复造成内存空间浪费，另一个是如果没有相应的清除或者数据管理的策略，运行的时间长了就可能出现内存泄漏的情况。当我们在使用redis这样的缓存的时候，实际上就是享元的思想。

<!-- more -->


&emsp;&emsp;在jvm的设计中，也体现了享元的影子。在jvm中有一部分区域叫做`常量池`，这部分区域存储了一些程序中的变量，以便在使用到相同的变量值的时候取用。字符对象有一个方法`intern()`，他返回一个相同的字符对象，输出结果相同，但是在jvm中却做了许多的操作，首先它会在调用的时候检查字符串常量池中时候存在，如果存在就直接返回字符串的引用，如果不存在，就将这个字符串的值添加到常量池中，然后返回这个字符串的引用。所以存在常量池中的两个字符串比较的话 `==` 是成立的因为 `==`都知道是比较的地址值，而常量池中相同值的字符串只有一个，地址值就是相同的。类似 `String str = "a";`这样的声明会将字符串放入常量池，而使用 `new` 创建的对象则不会，可以使用`intern()`来将其放入。还有就是在1.8及以上的jdk中，数值在-128-127的Integer，即使是使用 `new`的方式创建，`==`的结果也是成立的，这个题经常会拿来坑人。
&emsp;&emsp;那么在一个单体系统中，如何使用享元来为我们节省内存空间呢？类似的redis这样的缓存工具就是帮我们维护了一些数据结构，然后将数据个应用分离。而这些数据都是`k-v`格式的，这样的数据结构我们首先想到的应该是`Map`。使用map来存储共享对象，就是享元的核心了，至于取出来如何使用，那都是它的附加值；

首先我们看一下一个完整享元模式的成员：
1. 管理共享数据的工厂
2. 享元抽象类
3. 享元共享的数据的实现类
4. 享元非共享的数据实现类

先来介绍<2>, 他是享元操作的抽象，因为需要一类相似的对象；<3>是定义了这类对象中可以共享的数据该如何操作；<4>是这类对象中不能共享的对象该如何操作；<1>管理可以共享的这部分数据；
&emsp;&emsp;相应的我们来想象一个场景。2017-2018年，我们见证了共享单车的兴起和衰落。共享事物的出现，改变了我们的生活的同时，也用血的教训告诉我们，实现共产主义的前提，是人类精神的极大丰富...但是毕竟商业模式和共产还是有区别的，总之，如果每个人把这些共享单车稍有爱惜，规范停放的话，也许还是有可能成功的。
&emsp;&emsp;那么在这个场景中，我们骑车上班，需要用单车，而骑车这个动作是抽象的，骑共享单车上班是共享的实现，骑自己买的单车上班是非共享的实现；那么共享单车的管理者就是管理共享数据的工厂；
``` java
// 抽象自行车
@Data
public abstract class AbstractBick {

    // 车辆生产id （这个是车固有的，属于内部状态）
    private String bickId;
    // 骑车的人 （这个不是车固有的属于外部状态, 外部状态需要从外部获取）
    private String userName;

    public Bick(String bickId) {
        this.bickId = bickId;
    }

    // 自行车的骑方法的抽象
    abstract void ride(String userName);
}

// 共享单车的实现
@Data
public class SharedBick extends AbstractBick {

    // 共享单车的车锁
    private Boolean locked;

    public SharedBick (String bickId) {
        super(bickId);
        this.locked = true;
    }

    @Override
    public void ride(String userName) {
        this.userName = userName;
        if (locked) {
           System.out.println(userName + "非法骑车！！！"); 
        } else {
            // 只要锁开的就能骑，即使是别人开的锁
            System.out.println(userName + "开始骑车。车辆id: " + this.bickId);
        }
    }

    public void unlock() {
        this.locked = false;
    }

    public void lock() {
        this.locked = true;
    }
}

// 自己的自行车的实现
public class SelfBick extends AbstractBick {

    // 车辆购入的时候车的用户就是购买者
    public SelfBick (String bickId, String userName) {
        super(bickId);
        this.userName = userName;
    }

    @Override
    public void ride(String userName) {
        if(this.userName.equals(userName)) {
            System.out.println(userName + "开始骑车。车辆id: " + this.bickId);
        } else {
            System.out.println(userName + "正在使用车辆：" + this.bickId + ", 请确认是否盗用!");
        }
    }
}

// 共享单车管理者
public class SharedBickManager {

    private Map<String, SharedBick> bickMap = new HashMap<>();

    // 一开始区域单车管理员拥有的单车
    public SharedBickManager(Map<String, SharedBick> bickMap) {
        this.bickMap = bickMap;
    }

    public SharedBick getSharedBick(String bickId, String userName) {
        // 如果用户扫码骑车， 车必定是在管理车辆之中的；
        if (bickId != null) {
            SharedBick bick = bickMap.get(bickId);
            bick.unlock();
            return bick;
        }
        // 如果用户找了半天找不到车辆，上报id想骑车，管理就会加派车辆到用户小区（管理者应该也要分析是否有投放必要）
        if (bickId == null) {
            // 记录用户诉求分析,这里假设直接派车辆
            String bickId = UUID.randomUUID().toString();
            SharedBick bick = new SharedBick(bickId);
            bick.unlock();
            bickMap.put(bickId, bick);
            return bick;
        }
    } 
}
```
然后我们模拟一下使用的时候的状况
``` java
public static void main(String[] args) {

    // 单车管理者出现带着他的单车
    Map<String, SharedBick> map = new HashMap<>();
    map.put("111", new SharedBick("111"));
    map.put("222", new SharedBick("222"));
    map.put("333", new SharedBick("333"));

    SharedBickManager sm = new SharedBickManager(map);
    
    // 骑车的人扫码
    AbstractBick sb = sm.getShardBick("111");
    sb.ride("user1");

    // 骑自己车的先要买车
    AbstractBick sfb = new SelfBick("321", "user321");
    sfb.ride("user321");

}
```
可以看到一个完全的享元模式拥有这些元素，管理共享元素的类，维护了共享数据；抽象的享元类，内部定义了内部状态（不会改变的固有属性）和外部状态（随着外部改变的属性，要求从外部传入）；和享元的实现类（包括共享的实现，和非共享的实现）；但是使用的时候，非共享的实现是可以不存在的，内部外部状态也不是必须的。享元的主要思想就是复用相似的对象，节省空间，如果想要了解的话，首先可以从分析线程池的源码开始。