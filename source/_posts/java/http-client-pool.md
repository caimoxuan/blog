---
title: httpclient连接池处理及分析
date: 2019-12-13 23:12:09
thumbnail: 
categories:
    - java
tags:
    - experience
---


## httpclient的连接池

### 为什么要使用httpclient连接池

连接池是为了复用连接而存在的，就像线程池一样，创建了的线程在执行完成任务后不销毁，而是放入池中待命，以便执行下次任务的时候可以直接从池中取出线程执行，而不是先创建线程再执行，省去了创建线程带来的开销和时间。http连接也是一样的思路，http1.1支持了`keep-alive`，我们再对同一个网址进行请求的时候，就可以不用每次都建立连接；
我们都知道，http建立连接的过程是比较繁琐的，要经历3次握手和4次挥手，那么省去这个建立连接的过程在高并发的时候就会比较的有必要；同时，类似线程池，总有一个最大的线程数，和线程失效时间，因为在任务空闲的时候，这些空闲线程占用系统资源，所以我们要释放空闲时间长的线程。同样对于http连接，使用的是tcp的长连接，但是长连接的持有是非常耗资源的，特别是对于服务端，链接数是有限的，所以我们同样需要释放一定时间空闲的连接；

<!-- more -->

### 什么时候使用httpclient连接池
对自己的系统有正确的预估：
1. 调用的请求是否对同一个host大量请求；因为只有对于同一个host，长连接才可能建立，如果每次请求一会一个http://a.com 一会一个 http://b.com 这样是起不到连接池的效果的，并且http/https也是不能共用的，因为他们的端口不同；
2. 是否请求会达到一定的量级；如果请求的数量不高，连接池体现不出多大的效果；如果请求达到一定的量级，可能成为系统瓶颈或有较大影响的时候，可以尝试使用；
3. 对请求的系统有一定的了解；长连接是双向的，客户端维护连接，服务端同样需要维护连接，并且服务端服务的可能不止一个客户端，就怕到时候这边使用的连接池把服务端的连接占满了，导致服务端无法为其他的客户端提供服务，这个锅可能会扣在自己的头上。

### 如何使用httpclient

httpclient给我们提供了`PoolingHttpClientConnectionManager`这个类帮助我们来管理连接（版本4.5及以上）。在我们使用httpclient的时候，如果配置了这个连接管理，那么就会通过这个来按host管理连接；
还是一样，最简单的用法先来一个使用单例的：
``` java
public final class HttpClientUtils {

    private static CloseableHttpClient client;

    public static CloseableHttpClient getHttpClient() {

        if(client == null) {
            synchronized(HttpClientUtils.class) {
                if(client == null) {
                    // 先设置http连接的一些配置
                    requestConfig = RequestConfig.custom()
                                    // 从连接池获取连接的超时时间
                                    .setConnectionRequestTimeout(3000)
                                    // 建立连接的超时时间
                                    .setConnectTimeout(3000)
                                    // 请求的超时时间
                                    .setSocketTimeout(3000).build();

                    // 有的地方会配置一堆的https ssl的策略，点进这个构造函数他默认已经配置了，所以不用再配；
                    PoolingHttpClientConnectionManager manager = new PoolingHttpClientConnectionManager();
                    // 配置最大的连接数
                    manager.setMaxTotal(300);
                    // 每个路由最大连接数，路由是根据host来管理的，所以这里的数量不太容易把握；
                    manager.setDefaultMaxPerRoute(20);
                    client = HttpClients.custom().
                            setConnectionManager(manager).
                            setDefaultRequestConfig(requestConfig).
                            build();
                }
            }
        }
        return client;
    }

}
```
这样每次要使用http请求的时候，从这个工具中获取客户端来使用
``` java
HttpClientUtils.getHttpClient(); 
```

为什么要使用单例呢？其实直接`HttpClients.custom().setDefaultRequestConfig(requestConfig).build();`同样也是使用了连接池的，可以看build中的源码，没有设置manager的时候默认有一个。既然要管理连接池，那么这个管理器就只能有一个，不然管理就乱了，所以我们在使用的时候，只需要要一个`CloseableHttpClient`，这个client配置一个连接池的管理，每次使用都去找他才能达到连接管理的目的，否则这样写：
``` java
public static CloseableHttpClient getHttpClient() {
    return HttpClients.custom().setDefaultRequestConfig(requestConfig).build();
}
```
每次使用的时候都新建一个客户端，每个客户端是独立的，这样的话每次使用都完全是从头来一次。就好像使用线程池的时候，每次都是一个新的ExecutorService。

### 我见过的连接池配置人家写了一堆这里就这？

的确我见过人家配置了一堆的东西：
1. https/http协议策略，这个在`PoolingHttpClientConnectionManager`的空参数构造函数中就有；除非自己需要一些高级的自定义，否则不用重复添加；
2. `HttpRequestRetryHandler`配置的重试策略，对于个人来说像这样的请求不太喜欢重试，失败自己处理比较好；
3. 有一个定时任务的线程定时检测超过多长时间空闲的连接并销毁；

对于第三点，当我看到人家自己实现的定时检测任务的时候，我就在想，一个成熟的框架，人家不知至于想不到这点，那么就去看看它到底有没有做这件事。果然，框架的确考虑到了，但是这个默认没有开启，先看源码：在类 `HttpClientBuilder`中 
``` java
if (this.evictExpiredConnections || this.evictIdleConnections) {
    final IdleConnectionEvictor connectionEvictor = new IdleConnectionEvictor((HttpClientConnectionManager)connManagerCopy, this.maxIdleTime > 0L ? this.maxIdleTime : 10L, this.maxIdleTimeUnit != null ? this.maxIdleTimeUnit : TimeUnit.SECONDS, this.maxIdleTime, this.maxIdleTimeUnit);
    closeablesCopy.add(new Closeable() {
        public void close() throws IOException {
            connectionEvictor.shutdown();

            try {
                connectionEvictor.awaitTermination(1L, TimeUnit.SECONDS);
            } catch (InterruptedException var2) {
                Thread.currentThread().interrupt();
            }

        }
    });
    connectionEvictor.start();
}
```

可以看到，在`evictExpiredConnections` 或者 `evictIdleConnections`其中一个属性是true的时候，就会开启定时检测关闭连接的任务，那么我们就可以使用这样的方式来开启它：
``` java
client = HttpClients.custom().
            setConnectionManager(manager).
            setDefaultRequestConfig(requestConfig).
            // 简单开启
            evictExpiredConnections().
            // 如果还要自定义超时时间(可以看到它默认的是10s)
            evictIdleConnections(30L, TimeUnit.SECONDS).
            build();
```

`重要的一点是，如果开启了它的定时检测连接超时，那么使用的httpClient就必须是单例的，因为它创建一个httpClient就会开启一个线程检测，如果不是单例的，创建了多少的httclient，就会创建多少个检测的线程，这样和容易导致内存泄漏（可能是必然导致）`;

关于这些东西的使用，官方文档也有描述，但是描述的时候，他不会和你解释为什么，所以有的时候，看看源码就能理解它为什么要这样做以及他是如何做到这些的，httpclient的配置可不止这些，如果使用的话可以先看看它有没有帮我们实现，没有的话再去自己做；