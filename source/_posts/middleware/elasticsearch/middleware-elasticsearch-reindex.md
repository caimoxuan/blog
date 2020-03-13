---
title: elasticsearch修改自定义分词后重新索引
date: 2019-12-22 00:11:32
thumbnail: /gallery/thumbnail/elasticsearch.jpg
categories:
    - 中间件
tags:
    - elasticsearch
---

## elasticsearch使修改的自定义分词生效

在使用elasticsearch的时候可能会有这样的情况发生，现在index中有几千万的数据，由于一些查询显示的效果不佳，一些特殊的分词没有命中导致排序不理想。这个时候我们可以修改分词器中的自定义分词字典，这样就能让一些特殊的分词命中而提升score来让记录的排序上升。

<!-- more -->
首先这里用的使ik分词器，安装在elasticsearch目录下的plugin目录下，其中ik的配置文件中可以定义自己的分词字典，当我们修改了分词字典的时候，这个时候里面的记录是没有按新的分词建立过反向索引的，所以我们要做以下步骤：
1. 重新启动elasticsearch（启用新的字典）
2. 使匹配字典中修改的值的记录生成新的反向索引

重启这一步好做，杀死进程再启动就完成了，但是重新建立这些匹配词的索引就比较麻烦：
1. 重新导入数据
2. 修改匹配数据的分词字段来更新反向索引

第一种做饭简单粗暴，就是全量重新导入，但是对于大量数据来说会比较的慢，也容易出错；
这里说说第二种，原理就是更新document的一个field，让他对这个field重新索引，这里我们使用elasticsearch的`update_by_query`来完成：
``` bash
POST twitter/_update_by_query
{
  "script": {
    "source": "ctx._source.fieldName = ctx._source.fieldName", //这里让一个field = 自己，来达到更新字段的目的
    "lang": "painless"
  },
  "query": {
    "term": {
      "fieldName": "keyWords" // 这里只更新匹配的记录
    }
  }
}
```
执行上面的命令可能会比较慢，看匹配的数据的体量了；对于匹配的少量数据来说这样的方式还是比较方便的；如果有更多的需求，可以参阅官方文档：
https://www.elastic.co/guide/en/elasticsearch/reference/7.3/docs-update-by-query.html