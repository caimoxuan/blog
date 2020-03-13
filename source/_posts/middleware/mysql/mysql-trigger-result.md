---
title: 记录一下mysql的触发器创建和结果列合并
date: 2019-02-12 1:50:34
thumbnail: 
categories:
    - 中间件
tags:
    - mysql
---

## mysql中的结果合并和触发器

### 触发器：
``` sql
CREATE event IF NOT EXISTS count_sum_event ON SCHEDULE EVERY 10 MINUTE   
ON COMPLETION PRESERVE   
DO insert into xxx (l1,l2,l3)values(v1,v2,v3);

```
创建一个触发器并且每10分钟执行一次

SCHEDULE EVERY 10 MINUTE   10分钟执行一次

在每天的特定时间执行可以使用：（每日凌晨1点执行）

SCHEDULE EVERY 1 DAY STARTS DATE_ADD(DATE_ADD(CURDATE(), INTERVAL 1 DAY), INTERVAL 1 HOUR)


### 结果合并

结果合并有2种合并的方式：

1. 每个查询有多条的结果，需要将这些结果汇总到一个结果集种
这种是行合并，A结果种2条结果，b结果中3条结果，合并成5条最终结果，这个时候如果表字段相同可以用`union`，如果表字段不同的话，就用`union all`, 前提是原结果集中的列数要相同；
``` sql 
select id, user_name from t_user_1 union select id, user_name from t_user_2;

select a, b from t_tmp_1 union all select c, d from t_tmp_2;

```

2. 如果是多个查询，每个查询中选字段，然后将字段合并成单条结果
比如一个统计查询sql，查询总量，特定人群量，然后汇总：
``` sql
SELECT
lst_uv,
lst_pv,
lst_leader_uv,
lst_leader_pv,
CURRENT_DATE AS dt
FROM
	(SELECT
		count( 1 ) AS lst_pv,
		count( DISTINCT user_id ) AS lst_uv,
		CURRENT_DATE AS dt
	FROM
		gxt_user_access_details
	WHERE
		DATE_FORMAT( create_time, '%Y-%m-%d' ) = CURRENT_DATE
	) AS a,
	(
	SELECT
		count( 1 ) AS lst_leader_pv,
		count( DISTINCT ad.user_id ) AS lst_leader_uv,
		CURRENT_DATE AS dt
	FROM
		gxt_user_access_details ad
		LEFT JOIN gxt_user_corner uc ON ad.user_id = uc.user_id
	WHERE
		DATE_FORMAT( ad.create_time, '%Y-%m-%d' ) = CURRENT_DATE
		AND uc.is_provincial_leader = 1
	) AS b
WHERE
	a.dt = b.dt

```

查询几个指标并且将几个指标的值返回成一条结果（应该还有别的写法）；

### 插入查询结果从查询结果中（upSert功能，即存在更新；不存在插入）

``` sql

INSERT INTO gxt_user_access_summary ( lst_pv, lst_uv, lst_leader_pv, lst_leader_uv, dt )
-- 这里将上一个sql带入
on DUPLICATE key update lst_uv = a.lst_uv,lst_pv = a.lst_pv, lst_leader_uv = b.lst_leader_pv, lst_leader_uv = b.lst_leader_pv;

```

当然这个是需要唯一索引或者主键的，其中的dt就是唯一索引，代表的每日日期；然后使用`on duplicate key update` 来对主键或者唯一索引冲突的数据经行更新； 
