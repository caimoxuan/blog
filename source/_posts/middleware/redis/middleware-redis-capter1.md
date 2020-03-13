---
title: redis入门篇
date: 2019-11-27 14:02:07
thumbnail: /gallery/thumbnail/redis.jpg
categories:
    - 中间件
tags:
    - redis
---

## 不得不用的redis

&emsp;&emsp;现在的服务，基本上都是将功能封装成接口暴露出去提供restful风格的访问方式。当使用者调用接口的时候，除了接口的响应内容之外，还有接口的响应时间。当产品经理提出一个类似：评论中添加`点赞`和`取消点赞`功能，我们可以简单的完成：用户点赞就将数据库中的评论赞数加一，再次点击就减一。产品经理是不关心程序是如何实现的，只关心产品是不是符合预期。这样交付的产品，简单的测试是没问题的，如果没有压测的环节，到了线上才会真正的产生问题。
&emsp;&emsp;对于上述的需求，我们也可以稍加考虑的完成：用户点赞，在redis中用hash（这个hash表专门来做点赞的评论暂存）存入一个以评论id为键值为累计点赞数的记录，然后`inc1`,取消点赞就将这个键`inc-1`,然后定时将redis中赞的数据增加到数据库中的评论记录中。实际中还需考虑热点博客评论缓存，不能什么都向redis里塞。
&emsp;&emsp;那么就会问了，简单实现不是可以完成吗，为什么还要这么麻烦呢？简单的实现，对于访问量不大的时候来说是没有问题的，但是一旦用户增长，这样就会出现瓶颈，点赞返回变慢，体验变差，甚至点赞卡死。产品经理要做的是将产品的交互，功能等东西具体化，而作为开发者，我们不仅仅是实现产品，更重要的是知道如何让产品更稳定。如果说实现产品是武学外功的话，那么懂得优化产品就是武学的内功，绝世高手可不能不修内功吧。用好redis，能极大的提升接口的效率，帮助我们实现更稳定的系统。在这个数据爆炸增长的今天可以说是不得不用的。

<!-- more -->

## 主要功能

那么redis有哪些功能不得不用呢
1. 键值对存储，查询效率和tps较mysql极大的提高，首先快就是最好的功能；
2. 可设置的key过期时间，对于一些定期失效的数据（验证码）操作十分方便；
3. 数据持久化，较memcached来说数据更加的安全稳定（重启数据失效可能会造成短暂的缓存穿透）。
4. 丰富的数据结构，方便操作各种数据（经纬度坐标也是可以的）。
5. 帮助我们解决分布式架构中的各种问题（分布式限流、id递增生成、分布式session）。
暂时想到这么多，之后想到了再吹...

## 使用场景

在使用redis的时候，一般我们用于哪些场景呢？我们更具redis中的数据结构，这里简单举几个例子：
1. String （普通字符串） : 数据的缓存、键计数器、接口限流、分布式锁、分布式id生成、分布式session等
2. Hash （类似HashMap） : 暂存一类需要定时执行的任务数据（hash方便按类取出）方便按键值来修改个删除、缓存需要按键取的多字段的记录等；
3. List （类似LinkedList）:  简单列表，方便遍历，如一些商品数据等；
4. Set (类似HashSet) : 不含重复元素，无序，适合检查元素是否存在于缓存中，并且方便集合的`交并差`操作，缓存关系比较合适；
5. Zset (可以按分值排序的Set) : 通过一个score来排序，适合做排行榜(按榜记录大小排序)，评论列表(按时间排序)；
6. redis还有一个功能叫做`发布订阅`，这个功能可以代替简单的消息队列，但是没有堆积功能，而且不够可靠（好像UDP）,可以做操作日志同步；


## 数据结构

redis是使用c语言编写的，在redis中是如何实现这些数据结构的呢:
1. String : SDS - simple synamic string - 支持自动动态扩容的字节数组，和存储的数据有关，可以是 int(动态调整大小的数字小的时候4字节，大了变为8字节)、embstr（小于39字节的字符串）、raw（大于39字节的字符串），这些类型只是表象。
2. List : redis实现的quicklist,结构定义如下
``` c
typedef struct quicklist {
    quicklistNode *head;        // 指向quicklist的头部
    quicklistNode *tail;        // 指向quicklist的尾部
    unsigned long count;        // 列表中所有数据项的个数总和
    unsigned int len;           // quicklist节点的个数，即ziplist的个数
    int fill : 16;              // ziplist大小限定，由list-max-ziplist-size给定
    unsigned int compress : 16; // 节点压缩深度设置，由list-compress-depth给定
} quicklist;
```
而quicklist每一个节点又是一个ziplist（让每个节点都具有可压缩特性）

可以看到有头尾的指针，可以快速的从头或尾插入元素；有列表长度，获取长度复杂度O(1);

3. Hash :  常见的哈希表
当hash中数据不多的时候，hash中的数据用ziplist来表示，当超过一定阈值的时候，转换成hashtable。
hashtable这种数据结构很常见，键值对的表格，但是非常的占用空间，好处是通过key取value时间复杂度是O(1);

4. Set : 
如果熟悉java，可以发现java中的Set的实现是用了Map的，redis这里也是一样，使用了map来实现，只有一个键，没有对应的值，使用键的hashcode来定位和去重，所以set相比list可以快速的判断是否有相同的元素；


5. Zset : 
他的底层结构是ziplist或是skiplist，同样的，当数据超过阈值的时候转换为skiplist；
相比较Set，Zset提供了按一个记录（score）排序的特性，在插入的时候指定scorer，redis自动按score来排序。

## 操作命令

关于redis的操作，官方网站也有详细的介绍，一般来说使用redis都不会直接使用redis的操作命令，各个语言都会封装号对redis操作处理的客户端，对于java来说，如果不使用spring，可以使用jedis客户端来连接redis；如果使用了spring，可以使用spring-redis提供的redisTemplate操作redis的各种数据结构：
redisTemplate.opsForValue() 操作字符串
redisTemplate.opsForSet() 操作Set
redisTemplate.opsForZset() 操作Zset
redisTemplate.opsForList() 操作list
redisTemplate.opsForHash() 操作map
...
还有操作Geo(地理坐标)，hyperlogLog(bit存储基数)等等；