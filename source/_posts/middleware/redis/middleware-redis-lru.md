---
title: redis的淘汰策略和lru算法
date: 2019-01-01 7:24:57
thumbnail: /gallery/thumbnail/redis.jpg
categories:
    - 中间件
tags:
    - redis
---


## redis中的数据淘汰及lru算法分析

### 为什么需要淘汰数据

&emsp;&emsp;我们使用redis的时候，这个redis是有一个分配的内存大小的，我们的数据不可能无限的向redis种保存数据，redis可能会满，那么在redis满了的时候，再向redis中添加数据，就需要一个策略来处理接下来要发生的事。
在reids中我们可以通过配置 `maxmemory_policy` 来让redis选择策略淘汰key；

<!-- more -->

### redis的数据淘汰

redis中有6种数据的淘汰策略；
1. noeviction
如果超过最大内存，返回错误；
2. allkeys-lru
对所有数据使用lru算法淘汰；
3. volatile-lru
对设置了超时时间的key使用lru算法淘汰
4. allkeys-random
对所有的key随机淘汰；
5. volatile-random
对设置了超时时间的key随机淘汰；
6. volatile-ttl
根据设置的超时时间的时长进行淘汰（淘汰快要到时间的key）；

### lru算法

上述的几种淘汰策略，其他的理解起来都没有什么难度，除了这个lru算法。那么什么是lru算法呢？lru是 `Least Recently Used` 的缩写，就是一种能够快速计算出短时间内使用频次，这样我们就能知道哪些key在短时间内使用的频次会比较少；

下面列出几种常用的lru算法：
1. 链表
原理： 使用一个链表保存数据，新的数据插入链表的头部。当数据被访问的时候将数据提前到链表的头部，当链表满的时候，将链表最后的数据丢弃；
分析： 不需要额外的空间，在存在热点数据的时候这种算法的命中率会比较高，但是如果是周期性的访问，可能会造成真正需要保留的数据丢弃， 造成缓存污染；在访问数据的时候，需要遍历链表的方式将访问的数据放置到头部；

2. HashMap + 双向链表
原理： 使用HashMap保存key，value保存的是双向链表种的node节点，还是一样的在访问数据的时候将数据放置到链表的头部，这里优化的是取出数据可以从hashMap中快速的取出，然后放置到头部。在淘汰的时候也是将尾部的数据淘汰；
分析： 额外使用hashMap来保存数据，增加了空间的占用；但是好处就是取出的效率高了，典型的空间换时间。但是同样是存在链表一样的存在处理缓存污染的问题；

3. lru-k
原理：其中k代表的是访问到k次的时候做处理，使用2个列表，一个保存还没有访问到k次的数据（l1），一个保存已经到k次的数据（l2）。当数据没有到达k次访问的时候存在于l1中，当被访问到k次的时候从l1中移动到l2中，l2按照时间（到达第k次访问到当前的时间）排序，如果在l2中的数据再次被访问，就更新排序；淘汰的时候淘汰l2中排序后的在最后的数据（到k次距离当前最长的开始），一直在l1中的数据按照FIFO或是lru的方式进行淘汰。
分析： 不能完全解决缓存污染的问题，降低了影响。复杂度比较高，还要按时间排序，cpu消耗增加。实际应用中得出k在取值为2的时候效果会比较好；

### redis的lru算法

redis中存储数据的时候定义了一个自己的结构：
``` c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:REDIS_LRU_BITS; 
    int refcount;
    void *ptr;
} robj;
```
其中有一个lru字段，当数据被访问的时候，会将这个lru字段更新成`server.lruclock`, 这个东西是redis系统的lru时钟，redis会对`server.lruclock`每隔100毫秒进行一次更新（可以看作是按时间递增）。最后就是一个评估，根据对象中的lru字段和系统`server.lruclock`字段的差值来计算一个热度，差值小的代表热度就高，所以就淘汰差值大的就行了。

