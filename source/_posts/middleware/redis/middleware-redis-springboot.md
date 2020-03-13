---
title: springboot中使用redis
date: 2019-12-12 20:14:01
thumbnail: /gallery/thumbnail/redis.jpg
categories:
    - 中间件
tags:
    - redis
---

## redis在springboot中使用示例

### 引入依赖

创建好的springboot工程，是引入了springboot的依赖的，继承springboot的子项目，其中使用的就是springboot工程的版本，对于springboot-1.x 和 2.x来说，对于redis的依赖是不一样的，否则redis连接可能会出错；这里以2.x为例，2.x直接将配置放置在了父工程的依赖管理中，我们直接使用下面的依赖：

``` xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```

### 配置redis

引入依赖完成了，但是需要使用redis，我们需要先有一个redis，如果没有的话，可以看看docker快速启动一个redis，这里假设已经有了redis，springboot提供了许多中间件的自动装配功能，引入了上面的依赖，redis的自动装配就会被扫描，然而初始化redis连接的条件是要配置了redis的连接，我们可以在工程的: application.properties(默认)/application.yml(推荐使用，手动改后缀)中配置我们的redis连接：
``` yml
spring:
  redis:
    host: localhost #redis服务地址
    port: 6379 #服务端口默认6379
    database: 0 # reids中的数据库编号
    jedis:  # 连接设置
      pool: # 连接池参数
        max-active: 40  # 最大活跃线程
        max-idle: 10    # 最大等待连接数
        max-wait: 3000  # 等待连接超时时间
```

### 开始使用

使用之前，首先先要保证自己的redis能够连接（可以使用redis-cli命令行或者redis-desktop manager查看）。接下来需要使用redis做一个最简单的使用示例，由于我们在项目中配置了reids，springboot的自动装配就帮我们注入了RedisTemplate， 我们可以直接操作RedisTemplate，使用其中的模板方法（设计模式中的一种）, 推荐使用`StringRedisTemplate`，如果是简单操作的话；
``` java
@Repository
public class RedisTest {

    @Autowried
    private RedisTemplate redistemplate;

    public void set(String key, Object value) {
        redistemplate.opsForValue().set(key, value);
    }

    public Object get(String key) {
        return redistemplate.opsForValue().get(key);
    }

}
```
这样可以进行测试，但是可能会出现一些问题，特别是在做转换的时候，那是因为们少了一些配置，如果使用StringRedisTemplate操作的话，简单测试是比较简单的；

### 使用配置

redis中的数据存储是需要序列化和反序列化的，所以我们需要配置和redis通信之间的序列化方案，否则，key可能无法正常命中， value可能不是预计的值；一般来说我们使用可能会存储json比较多，可以配置将redis的key的序列化方式配置成string类型的序列化，而value的序列化方式配置成json序列化，但是我个人不是很喜欢直接配置成json序列化，而是全都使用string的方式，由业务来处理json数据的转换;
``` java
// 主要就是RedisTemplate中的方法：
redistemplate.setKeySerializer(serializer); //设置key的
redistemplate.setValueSerializer(serializer); //设置value的
redistemplate.setHashKeySerializer(serializer); //设置hash数据中的key的
redistemplate.setHashValeSerializer(serializer); //设置hash数据中的value的

//对于serializer，提供有一下几种，也可以自行实现：
JdkSerializationRedisSerializer // jdk的序列化方案；
Jackson2JsonRedisSerializer //jackson 转json的方案；
StringRedisSerializer //字符串的方案 （目前最常用的方案）
... //还有几种不常用，也可以自己实现；
```

### 相关操作

在redis中提供许多种类的数据结构，在springboot中我们如何愉快的操作他们呢, RedisTemplate当然已经帮我们封装好了；
``` java

// 操作简单的字符串:
redistemplate.opsForValue();
// 操作List
redistemplate.opsForList();
// 操作 Set
redistemplate.opsForSet();
// 操作 Zset
redistemplate.opsForZSet();
// 操作hash
redistemplate.opsForHash();
... //以上是常用的，还有如地理坐标等的操作，使用到可以再看；

```
每种操作返回的是一个封装好的CURD的Operations，对应的是不同的方法操作；基本上和redis-cli中的操作都能由对应；

### 发布订阅

发布订阅是redis的一个附加功能，没有专业的消息队列那么强大，对于一些简单的数据分享场景可能也会使用到：
``` java
//消息的发送
redisTemplate.convertAndSend("topic", "message");

// 消息的订阅接收， 主要就是注册2个对象，可以放在一个@Configuration中
// 这个对象声明订阅的主题；
@Bean
public RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory, MessageListenerAdapter responseListenerAdapter) {
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.addMessageListener(responseListenerAdapter, new PatternTopic("topic"));
    return container;
}

//这个对象定义好消息如何处理， 就是自己定义一个对象，将对象和处理的方法名称作为参数传入；
@Bean
public MessageListenerAdapter loginListenerAdapter() {
    return new MessageListenerAdapter("这里传入一个处理器的类", "处理器中处理方法的名称");
}

```



