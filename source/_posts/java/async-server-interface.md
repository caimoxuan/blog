---
title: java异步处理http请求同步返回
date: 2019-12-09 17:02:01
thumbnail: 
categories:
    - java
tags:
    - experience
---


## 如何设计一个接口，使用消息队列异步请求，但是客户端同步收到结果

&emsp;&emsp;异步处理，同步返回？为什么会有这样一个需求？既然接口要求同步返回，那么直接阻塞就好了，要什么异步消息同步返回？互联网行业的请求有个特点，在特定的时候高并发，平常的时候流量平缓。而高并发保护系统的手段是缓存、限流、降级。限流有许多的手段，想令牌桶、漏桶算法按数量限流，也有使用消息队列，排队限流的。至于使用消息队列的好处就不多说了，这里主要将如何实现这个需求，有一个系统比较的不稳定，但是没人维护，又不能替换它，只能在他的上层加一层来保护她，可以限流处理，也可以用mq让它以他的最大处理能力处理。说白了这东西就是一个缓冲系统，可替代性高，存粹的技术型应用，由于新鲜所以我觉得可以一试；


<!-- more -->

### 选型

&emsp;&emsp;首先我们来选型，分析需求：使用消息队列异步请求，那么选型消息队列： zeromq、rabbitmq、activemq、kafka、rocketmq等等，消息队列很多如果没有什么要求，那么都可以选，但是首先我们需要考虑实现问题呢，使用的mq是否支持。我们需要可以排队，那么zeromq就不能选了，activemq有较小概率丢失消息，一般我不太爱用这个。好了我们实现这个需求不需要什么复杂的功能，那么剩下的都是可以选的，接下来就是考虑架设成本和易用性的问题。rabbitmq的时效性非常的好，但是吞吐量不及kafka和rocketmq，而且隔热你用的较少；所以一般来说我习惯在kafka和rocketmq中选择。rocketmq综合性能比较好，而且有很多的功能（消息提交重新消费、延时消费等），做支付金融首选rocketmq，但是我们这里不需要用到这些，所以这里用了kafka。
&emsp;&emsp;有了异步处理消息的mq，我们还需要一个保存mq处理完的返回值队列，能让阻塞的线程获取到。因为要分布式的，所以这个队列不能是java中的数据，所以这里使用redis保存mq处理完的数据。

### 架构

&emsp;&emsp;接下来我们先构造系统，首先我们有一个web服务，用来接收http的请求，接受请求后发送mq处理，然后阻塞当前处理的线程，等待mq处理完成，从redis的队列中取出数据，现在还差一个mq的接收方，实现一个server服务，接受mq消息并处理，然后将数据放入redis，并且通知web的这个线程消息已经处理完毕，让web这个阻塞的线程取出redis中处理完成的数据。至于通知需要广播通知，因为分布式的话这个处理请求的线程会在任意一台web服务中，至于这个通知我们可以用redis的发布订阅功能来实现；
&emsp;&emsp;整体我们就有2个服务，一个web，一个server，之间通过mq通信，redis共享数据，redis发布订阅同步状态唤醒线程。


### 实现

&emsp;&emsp;首先是web端的实现，简单的springboot项目加上web依赖，这里不赘述，这里我们模拟场景：我们需要去一个三方系统获取用户信息，通过后台http调用获取他的用户信息,用户要么输入手机，用户名或邮箱和密码（加密的）；
``` java
@RestController
@RequestMapping("/async")
public class AsyncController {

    @Autowired
    private AsyncRequestExecutorService asyncService;

    /**
     * json处理工具 这里用的gson 也可以fasejson或其他
     */
    private Gson gson = new Gson();

    @RequestMapping(value = "/userInfo", method = RequstMethod.POST)
    public String getUserInfo(UserInfoQueryDTO userInfoQuery) {
        //正常的话我们调用一个service就得到返回值就同步返回了，客户端就能获取到相关信息；但是这里要异步处理，同步返回，我们准备一个异步处理的线程池来处理；先看AsyncRequestExecutorService;
        try{
            Future<String> result asyncService.doRequest(UUID.randomUUID().toString(), gson.toJson(userInfoQuery));
            // 这里需要设置超时时间， 保证能给客户端一个反应(这里就实现了阻塞)
            return result.get(5， TimeUnit.SECONDS);
        } catch(Exception e) {
            //这里要么超时要么失败处理
        }

    }


}

@Data
public class UserInfoQueryDTO {

    private String mobile;

    private Strig userName;

    private String email;

    private String password;
}

@Service
public class AsyncRequestExecutorService {

    /**
     * 线程名称方便定位问题
     */
    private String final threadName = "ASYNC-THREAD-"；

    /**
     * 这就是kafkaspringboot的简单集成使用kafkaTemplate的发送消息这里不详述
     */
    @Autowired
    private KafkaMessagePublisher messagePublisher;

    /**
     * redis的集成，保存mq处理完成后的结果集，这里用的是set数据结构，因为这里请求时一次性的 返回就移除pop()方法正好满足， 而且请求不重复
     */
    @Autowired;
    private ResponseRedisCache responseCache;

    /**
     * 线程池
     */
    private ExecutorService executorService;

    public AsyncRequestExecutorService() {
        this.executorService = new ThreadPoolExecutor(Runtime.getRuntime().availableProcessors() * 8,
                200,
                3000,
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(),
                new ThreadFactory() {
                    private AtomicInteger count = new AtomicInteger(0);
                    @Override
                    public Thread newThread(Runnable r) {
                        return new Thread(r, THREAD_NAME + count.incrementAndGet());
                    }
        });
    }

    /**
     * 实现异步处理的关键
     * @param requestId 当前请求的id 可以用uuid
     * @param message 当前参数（json格式）
     * @return 返回一个Future用来阻塞
     *
     */
    public Future<String> doRequest(String requestId, String message) {
        // 我们收到消息后发送mq处理(这里一定要将requestId一起处理，方便server处理完放入redis后的存取)
        messagePublisher.send(requestId, message);
        // 返回一个线程处理
        return executorService.submit(() -> {
            //一个静态map保存当前线程（注意使用ConcurrentHashMap）
            GlobalThreadMap.parkThreadMap.put(requestId, Thread.currentThread());
            // 立即阻塞，应为它不会马上处理完成的(最多阻塞5s)
            LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(5));
            // 之后是线程被唤醒的处理
            // 首先从静态map中删除当前请求的线程
            GlobalThreadMap.parkThreadMap.remove(requestId);
            // 返回从redis中取出的结果，是null也直接返回(因为5s的阻塞时间，过了5s还没处理完就需要响应客户端了，但是这时redis还没有数据)；
            return responseCache.pop(requestId);
        })    
    }

}
```

以上是web端的处理，是关键部分，接下来是server端的处理，比较简单，就简单叙述下：
在web中向kafka中推送了一条获取用户信息的消息，接下来就只要处理一下步骤：
srver端消费消息
反序列化
http调用第三方，同步获取返回结果（这里注意配置http调用的超时时间和异常处理）
将http的返回结果用消息中的requestId作为key写入redis
最后通过发布订阅返回requestId处理完成的消息

到这里这条请求的处理又回到了web端：
web端收到了redis的发布订阅消息，从`GlobalThreadMap`中用发布订阅的requestId（也就是一开始的UUID生成的id）取出被park的线程执行unpark唤醒，之后`result.get(5, TimeUnit.SECONDS)`就能获取从redis中取出的数据完成一次请求处理；

### 总结

1. 使用kafka异步处理请求，注意消费端消费请求的时候用多线程处理。
2. 使用Future阻塞处理的线程达到同步返回的目的。
3. 使用redis保存处理结果按请求id取出；发布订阅通知阻塞的线程处理完成（否则就要轮询队列来判断是否完成，浪费cpu）。
4. 其中使用到的kafka和redis都是springboot的整合，这个又大量教程。
5. 其中的线程数等优化还需结合实际，也许不会比直接限流强多少。