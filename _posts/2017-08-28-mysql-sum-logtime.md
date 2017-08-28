---
layout: post
title:  "Mysql统计在线时长"
date:   2017-08-28 13:36:22
author: coldxiangyu
categories: Mysql
tags: Mysql sql
mathjax: true
---

* content
{:toc}


上周，大学室友在群里发出了江湖救急，悬赏一个mysql数据库统计在线时长的sql，当然是免费悬赏。
作为室友，本人当然是义不容辞的。
首先，问他要了数据表结构和部分测试数据。如下：
``` sql
CREATE TABLE IF NOT EXISTS `qw_ht_tbl_player_login` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `userid` int(20) DEFAULT NULL,
  `type` tinyint(4) DEFAULT NULL,
  `channel` int(4) DEFAULT NULL,
  `ip` varchar(32) DEFAULT NULL,
  `device_id` varchar(128) DEFAULT NULL,
  `log_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `userid_key` (`userid`),
  KEY `time_key` (`log_time`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8 AUTO_INCREMENT=26573939 ;
```




其中id自增，userid为登陆用户，type通过1,2区分登录、登出，log_time为对应时间。
部分数据如下：  

![image_1boja9d801s3a1hmtqc81fp618qr9.png-26.7kB][1]  
首先我觉得这个表设计的不是很合理，不如一条数据一个start_time一个end_time来的方便，不过倒也蛮清晰。  

接下来就是大概思路，要做到获取某个用户登录的在线时长，肯定是由该用户的每次登出时间 - 该用户对应的登录时间，最后求和。  

登录时间和登出时间有很多，如何保证相邻的登出时间和登录时间相减呢？我们知道id是自增的，可以找到最小的id差值来确定对应的时间。

我们以下面两条数据为例：  
![image_1bojaqrbr1glb1sch1ri91oi0frsm.png-7.3kB][2]  
如何通过登录ID 2726768来确定登出ID 2729263呢？
sql如下：
``` sql
select min(id) from qw_ht_tbl_player_login where userid='1129938' and type=2 and id > 2726768
```
查询结果：2729263

由此我们可以获取每一个时间差，进而求和，sql如下：
``` sql
select 
    sum(t2.log_time - t1.log_time)
from 
    qw_ht_tbl_player_login t1,
    qw_ht_tbl_player_login t2 
where 
    t1.userid=t2.userid and 
	t1.type=1 and 
    t2.type=2 and 
    t1.log_time <t2.log_time and
    t2.id=(select min(id) from qw_ht_tbl_player_login where userid='1129939' and type=2 and id>t1.id)
```

我们如何验证这条sql的准确性呢？
我单独update了8条数据，userid为1129938，其余仍为1129939不变，通过这几条数据来验证sql结果是否正确。  
执行`select * from qw_ht_tbl_player_login where userid = '1129938' order by id`，  数据如下：  
![image_1bojbnrcn13go1kfald315u9132q13.png-22.4kB][3]  
可以看到，一共有四次登录登出，也就是四个时间差值，分别为10s,5min41s,4min10s,2min51s,求和为772s。
将上面sql中的userid替换为1129938，执行结果如下：  
![image_1bojbvahcppiir01gi173pkt91g.png-3.9kB][4]  
与实际并不相符。
这时候就要找原因了。
首先我们逐个打印出每个差值进行排查，在哪里出了问题，如下：  
![image_1bojc2psb1q0s1rbdm8v1l41ins1t.png-40.2kB][5]  
看到这几个差值再对比实际差值，我意识到了自己的错误。  
time类型数据直接相减得到的结果并不是按时间类型相减的。
这时候，我改成按时间相减，再次打印结果对比如下：  
![image_1bojc9kc1siev2l4364g9ej02a.png-40.1kB][6]  
我们更改最初的sql，再次打印结果：  
![image_1bojcd6p8c0p1gch9fm3r31e7r2n.png-27.3kB][7]  
与实际预期一致。  
最终sql如下：
``` sql
select
    SUM(TIMESTAMPDIFF(SECOND, t1.log_time, t2.log_time)) as RESULT
from 
    qw_ht_tbl_player_login t1,
    qw_ht_tbl_player_login t2 
where 
    t1.userid=t2.userid and 
    t1.type=1 and 
    t2.type=2 and 
    t1.log_time < t2.log_time and
    t2.id=(select min(id) from qw_ht_tbl_player_login where userid='1129938' and type=2 and id>t1.id)
```
<br/><br/>
>版权声明：本文为原创内容，转载请注明出处，谢谢合作。

  [1]: http://static.zybuluo.com/coldxiangyu/rbmo47wc6syuz0ftwg85nuwm/image_1boja9d801s3a1hmtqc81fp618qr9.png
  [2]: http://static.zybuluo.com/coldxiangyu/z0tjh9wfrus6ljo0ylvf6ol5/image_1bojaqrbr1glb1sch1ri91oi0frsm.png
  [3]: http://static.zybuluo.com/coldxiangyu/qe46xh83bt68gw2z1ze6oh5t/image_1bojbnrcn13go1kfald315u9132q13.png
  [4]: http://static.zybuluo.com/coldxiangyu/oyox03o8ca4t3bilvid2l6y1/image_1bojbvahcppiir01gi173pkt91g.png
  [5]: http://static.zybuluo.com/coldxiangyu/4ol04f9prlghbzbnukgtqo56/image_1bojc2psb1q0s1rbdm8v1l41ins1t.png
  [6]: http://static.zybuluo.com/coldxiangyu/3i2fap8bqatc8znft8bhzk7f/image_1bojc9kc1siev2l4364g9ej02a.png
  [7]: http://static.zybuluo.com/coldxiangyu/r9x3ffvlpqcvd2kg8k032r6s/image_1bojcd6p8c0p1gch9fm3r31e7r2n.png
