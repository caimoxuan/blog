---
title: elasticsearch-入门篇
date: 2019-12-18 22:52:47
thumbnail: /gallery/thumbnail/elasticsearch.jpg
categories:
    - 中间件
tags:
    - elasticsearch
---

## elasticsearch概述

elasticsearch据说是起源于一个外国男子给妻子做的搜索食谱，结果发现自己的搜索越做越牛逼，然后耽误了给妻子的搜索食谱的故事... 忘了从哪里看来的。别的不多说，他的主要功能就是全文搜索，很大程度上，理解起来就是将mysql中不能或者是难做的`like`查询的方式给做到极致。

<!-- more -->

### elasticsearch能做什么

既然是全文检索，那么做搜索引擎是可以的。其次，很多依赖关键字的搜索都可以看到它的身影，类似快餐店搜索、企业信息搜索、文章搜索等等；你可以只提供关键字，由它来提供包换关键字的内容，当然关键字的描述程度决定了结果的准确性；这不经让我想到一个笑话：
一位女士来到药店给丈夫买药，但是忘记了药的名称。这时药店老板开口了：“以我买药20几年的经验，只要你说出其中的几个字，我就能知道你要买什么药！”。这个时候女士颤颤巍巍的说了2个字: “胶囊...”。
这样一个笑话在生活中也许是笑话，因为现实中医生也不能将所有的胶囊给列举出来，但是elasticsearch可以，只要存储的数据足够的话，能列出所有的胶囊，也许在看到药的名字的时候，就能想起来买的是什么药了。
还有也就是目前来说用的最多的功能，ELK，用来做日志分析和监控报警链路跟踪等功能。ELK涉及3个组件，E:elasticsearch、L:logstash、K：kibana；L是用来同步数据的，可以将服务器上的数据同步到E中，那么E就是存储数据，做全文检索；既然要分析，那就需要数据可视化，kibana就是用来做这个功能，他能将E中查询出来的数据以图形化的方式展示出来，当然这个查询是可定制的；

### elasticsearch的简单原理

说到elasticsearch的原理就不得不提到`Lucene`，elasticsearch是基于它的，他是一个高效的基于java的全文检索库。那么说明叫全文检索呢？前面并没有说明什么是全文检索。其实也很容易理解，对于非结构化的数据（mysql中的数据是结构化的），类似一大坨的日志、一篇文章等，想要通过某个关键字查询到含有关键字的文章。
一种做法是顺序扫描，类似于linux中的`grep`，时间复杂度（我自己瞎想的）应该是O(n)-n为字数。
还有一种做法就是类似新华字典，先索引，后查询。这种方式就是我们所说的全文检索。通过这种方式构建的索引，被称为反向索引（倒排索引）。
举个例子，我们有以下句子：
1. 今天天气真不错；
2. 今天是个好日子；
3. 明天天气会更好；
那么在数据插入的时候就会被做反向索引。首先，分词，一个好的分词是索引的关键，分词就是按生活中出现的词组进行分词，当然，中文的分词是很复杂的。上面的3个句子分成几个词语（这里的分词器就是我）：
``` bash
今天        1，2
天气        1，3
真不错      1
是个        2
好日子      2
明天        3
会更好      3
```
可以看到，上面就是模拟Lucene的做法，将3句话分成若干个词，每个词后面的是这个词出现的句子的id，类似这样的做法就是反向索引的创建过程。那么查询的时候，比如输入关键字 `天气真不错`, 那么首先这个关键字也会通过分词器分成`天气`和`真不错`。然后再通过反显索引查询出所在的句子 1，3 和 1 被命中，然后聚合成 1， 3的句子的id，就能通过这个id再去查询句子。


### 其他

1. 分片、副本

elasticsearch支持的副本和分片和mongodb是类似的，在elasticsearch中可以十分方便的监控集群的状态，使用head插件或者kibana可以在集群中看见三种状态：
green：所有分片和其副本运行正常；
yellow： 存在分片的副本运行不正常，所有分片正常；
red： 存在分片运行不正常，此时无法正常工作；

2. 概念类比

|  mysql | elasticsearch  |
|  ----  | ----  |
| database | index |
| table | type |
| row | document |
| column | field |
| schema | mapping |

3.  查询方式

采用restful的接口方式，推荐安装完成es之后下载elasticsearch-head插件，本地启动，也可以在elasticsearch中集成（启动es后es服务端口9200，head服务端口9100），本地启动注意需要在elasticsearch配置文件中配置`处理请求跨域`配置， 如果开启了x-pack还需要`允许请求头`配置
``` yml
http.cors.enabled : true
http.cors.allow-origin : "*" 
http.cors.allow-methods : OPTIONS, HEAD,GET, POST, PUT, DELETE
http.cors.allow-headers: X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
```

本地启动访问head地址如果开启了x-pack用户密码验证，注意访问的时候带上`?auth_user=${userName}&auth_pass=${password}`,这样请求连接elasticsearch的时候可以带上用户名密码

其他查询方式，创建index等参阅官方文档；

