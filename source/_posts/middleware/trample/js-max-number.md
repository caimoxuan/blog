---
title: js数字的最大值导致的传入后台参数不正确
date: 2019-02-07 15:11:23
thumbnail: 
categories:
    - 踩坑
tags:
    - js
---


## js中的number类型的最大值

js中的数字是有一个安全值的限制的，超过了这个安全值，就会出现不准确的现象：

在其他语言如java中Long可代表的最大值	   9223372036854775807 （2^63 -1）
在js中的安全值的限制是							  	9007199254740991 （2^53 - 1）

<!-- more -->

### 出现的问题：

1. 在后台传值到前端的时候，可能有的作为主键的值（id）会大于这个安全值，那么在展示以及想要更新条目传入后台的id会不一致导致修改无法完成或者是展示数值错误；

2. 前端使用一个超过安全值的字段作为number类型参数传入后台，那么这个字段就会改变（变成一个小于安全值的值），如果这个字段带有标识意义或者是查询的id，那么结果就会出乎预期；


### 处理方案

1. 前端注意：

前端在处理传值的时候，要注意如果字段长度可能超过安全值，那么应该用string传值，类似:

{"userId": 191231232132123144}  ->  {"userId": "19123123213212314"}

处理传值的时候将字段传为string

2. 后台注意：

在对前端传数据的时候，如果包含可能超过前端安全值的字段，可转为String，或者是将字段的值控制在安全范围内；