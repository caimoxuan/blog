---
title: 当mysql的left join和预期不符，才知道规范是多重要
date: 2019-01-10 22:24:25
thumbnail: 
categories:
    - 中间件
tags:
    - mysql
---


## left join居然出现了意料之外的结果

问题的描述：

有一张购买明细表 t_meeting_order 和一张 渠道表 t_channel，查询购买明细中 渠道id和渠道表中关联的渠道，这个时候我发现明细表里面的：channel_code字段和渠道表里面的channel_code是对应的，由于都不是自己建的表，没有仔细看表结构，随手就是一个`left join`

<!-- more -->

但是接下来让我懵逼的事情发生了，明细表中只有一条数据，渠道表里由3条数据，这3条数据的channel_code 都不相同，但是结果却出现了3条数据...

t_meeting_order 中的一条数据的channel_code是 `0` 而 t_channel 中的channel_code是 `x0x`、`x1x` 、`x2x`;
然后通过查询语句：
``` sql
SELECT
	m.channel_code,
	l.channel_code
FROM
	t_meeting_order m
	LEFT JOIN t_channel l ON m.channel_code = l.channel_code;
```

查询寻的结果居然是：
|m.channel_code|l.channel_code|
|---|---|
|0|x0x|
|0|x1x|
|0|x2x|

这完全就超出了个人的认知了，什么情况，难道这个left join 有什么知识点我没有掌握的吗，这不应该啊。先试试改改数据看看，首先把channel表中的一个值改成 0 试试？结果还是这样。那就再试试将order中的值改成 x0x 试试，结果发现了问题所在，这个时候报错了，原来 t_meeting_order 中的channel_code的类型是`int` , 而t_channel中的channel_code的类型是`varchar`。吐了,我们知道再查询的时候，如果查询的字段的类型where的条件字段是个varchar的类型，那么再查询的时候就需要加上引号，如果不加的化，就会出现不匹配的数据。
在本例中当中，当int类型的值为 0 ，而varchar是字符串 x0x 的时候，left join的on在连接的时候没有正确的匹配记录，这个原因没有仔细的往原理去研究，但是造成这个结果的主要原因就是规范不统一；类似这样的小问题有时候也会搞死个人哦；
这应该是个不成文的规范：
现在在mysql中不推荐使用外键，由业务来处理这些关联关系，那么没有了外键的强校验，在类似这样的具有关联关系数据的时候，相关联的字段应该统一类型、长度、名称和默认值。这样可以尽量的避免问题的发生；
