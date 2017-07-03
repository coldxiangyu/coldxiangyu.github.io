---
layout: page
title: Collections
permalink: /collection/
icon: bookmark
type: page
---

* content
{:toc}

## 架构设计

* [一些设计上的基本常识](http://javatar.iteye.com/blog/706098)
    
    注意在框架设计过程中，将API/SPI进行区分
    
* [如何建设高可用系统](http://ifeve.com/%e5%a6%82%e4%bd%95%e5%bb%ba%e8%ae%be%e9%ab%98%e5%8f%af%e7%94%a8%e7%b3%bb%e7%bb%9f/#more-31382)

    基本上能想到的都已经列出来了，在以后的高并发架构设计过程中，还是要回顾一下这些是否已经考虑
    
* [持续部署，并不简单！](http://www.open-open.com/solution/view/1433670703505)

    持续部署的方案以及应对场景
    
* [大型网站架构设计原理（书的目录）](http://www.cnblogs.com/powercto/p/6817556.html)

    虽然是目录，但能从中学到的东西很多
    
* [微服务架构的基础框架选择：Spring Cloud还是Dubbo？](http://blog.didispace.com/microservice-framework/)

    对Spring Cloud和Dubbo进行了全方位对比，也是目前YBT云核心Spring Cloud替换dubbo的一个参考

## 网络编程

* [RPC原来就是Socket——RPC框架到dubbo的服务动态注册，服务路由，负载均衡演化](http://www.cnblogs.com/intsmaze/p/6058765.html)

    虽然不是很严谨，但是能体现出整个RPC的思路。缺点：并没有使用动态代理的方式进行处理。
    
* [TCP/IP四层模型](http://www.cnblogs.com/BlueTzar/articles/811160.html)

    严肃的TCP/IP讲解
    
* [HTTP协议—— 简单认识TCP/IP协议](http://www.cnblogs.com/roverliang/p/5176456.html)

    配图，生动形象，逗比，易懂 

* [对 TCP/IP 网络协议的深入浅出总结](http://blog.jobbole.com/74795/)

    HTTP，TCP/IP，SOAP，SOCKET总结的大成，socket总结不足
    
* [关于RPC协议的通俗理解](http://blog.csdn.net/huangjin0507/article/details/52199349)

    总结了知乎同问题下精彩的回答，通俗易懂
    
* [HTTP、TCP、UDP、Socket、servlet区别与联系](http://blog.csdn.net/wabiaozia/article/details/54571874)

    说实话，这篇文章并没有什么亮点，而且并没有题目说的那样讲述区别和联系，都是介绍大概的概念完事了，收藏下来只是因为在讲Servlet的部分吸引了我
    
* [RPC框架几行代码就够了](http://javatar.iteye.com/blog/1123915)

    dubbo框架作者，不需要多说
    
* [Java Socket编程](http://www.cnblogs.com/wnlja/p/4366402.html)

    网络编程，基本上就是socket编程，而socket编程，这一篇就够了

## JAVA

* [Google Java编程风格指南](http://www.hawstein.com/posts/google-java-style.html)

    与alibaba的java编程手册相比，我觉得谷歌的相对来说更简洁一些

* [JAVA中分为基本数据类型及引用数据类型](http://www.cnblogs.com/dubo-/p/5565677.html)

    将java的数据类型讲解的很清楚，从计算机存储方式以及内存方面都有说明，不过在String同值判断的时候存在错误，忽略了java享元模式，需要特别注意
    
* [GBK,UTF-8,和ISO8859-1之间的编码与解码](http://blog.csdn.net/xiongchao2011/article/details/7276834)
    
    详细讲述了几种编码之间的区别，按历史发展顺序，并以java为例详细说明了关于编码设置与转换
    
* [java代理机制](http://www.cnblogs.com/machine/archive/2013/02/21/2921345.html)

    java几种代理模式以及实现方式讲解的很清楚  
    
* [关于.getClass()和.class的区别](http://blog.csdn.net/qianzhiyong111/article/details/7320879)

    反射机制
    
* [深入探讨Java的类加载机制](http://www.blogjava.net/William/archive/2006/08/25/65804.html)

    对JVM类加载机制，逐层加载讲解的比较到位
    
* [单例模式的优缺点和使用场景](http://www.cnblogs.com/damsoft/p/6105122.html)

    关于单例，有这一篇也就差不多够了，主要是单例的有点和缺点总结的比较全面
    
* [聊聊并发——生产者消费者模式](http://blog.csdn.net/emac/article/details/51622199)

    基于队列的生产与消费
    

## 框架方面

* [[Spring Boot 系列] 集成maven和Spring boot的profile功能](http://blog.csdn.net/lihe2008125/article/details/50443491)  
* [spring boot应用启动原理分析](http://blog.csdn.net/hengyunabc/article/details/50120001)  
* [Spring cloud中文文档](https://www.springcloud.cc/spring-cloud-dalston.html)  



## 算法

* [动态规划：从新手到专家](http://www.hawstein.com/posts/dp-novice-to-advanced.html)

    研究算法，动态规划必不可少
    
* [位运算的技巧及进阶](https://github.com/xuelangZF/LeetCode/tree/master/BitManipulation)

    在刷LeetCode的过程中发现的一篇，总结十分详细，也是我写[从头学算法（五、位运算）](http://www.coldxiangyu.com/2016/07/28/%E4%BB%8E%E5%A4%B4%E5%AD%A6%E7%AE%97%E6%B3%95-%E4%BA%94-%E4%BD%8D%E8%BF%90%E7%AE%97/)的参考之一
    
* [原码, 反码, 补码 详解](http://www.cnblogs.com/zhangziqiu/archive/2011/03/30/ComputerCode.html)

    对源码、反码、补码的讲解，一读就懂
    
* [LeetCode 刷题指南（一）：为什么要刷题](http://www.tuicool.com/articles/raAbEbq)

    在我刷leetcode焦头烂额的时候看到的。不要盲目刷题，要有目的的刷题
    
* [从入门到精通我是这样学习算法的](http://blog.csdn.net/lifuxiangcaohui/article/details/28642075#comments)

    大神是怎样学算法的

* [Java中hashCode的作用](http://blog.csdn.net/fenglibing/article/details/8905007)

    详述了hashCode的作用与实现，相关还有集群下的一致性hash
    

## redis相关

* [美团在Redis上踩过的一些坑](http://blog.csdn.net//chenleixing/article/details/50530419)

    美团技术负责人深入讲解使用redis过程中遇到的问题，重要的是寻找问题的过程，很有帮助
    
* [Redis在项目中实战](http://blog.csdn.net/u010539352/article/details/51787324)
    
    文章比较简单，主要讲解了在实际项目中作为缓存的应用以及应用场景，以及实现方法
    
* [使用Redis之前5个必须了解的事情](http://www.csdn.net/article/2014-09-29/2821930-5-key-takeaways-for-developing-with-redis)

    盲目使用redis的事先一定要设计好redis存储数据类型以及命名方式，省的自己都不知道存了些什么，不要有个锤子看哪都是钉子
    
* [浅谈Redis数据库的键值设计](http://www.searchdatabase.com.cn/showcontent_52657.htm)
    
    此文甚好，虽然是python版的，但是讲的很清晰
    
* [最常用的缓存技术---redis入门](http://www.cnblogs.com/fengru/p/5793087.html)
    
    对redis各种数据类型讲解的很透彻
    
* [Redis百亿级Key存储方案](http://www.cnblogs.com/colorfulkoala/p/5783556.html)

    评论很好玩
    
* [KEY设计原则与技巧](http://www.cnblogs.com/nixi8/p/6708252.html)

    很少见的通过关系型数据库表对比改造为redis的数据结构
    
* [Redis实现类似SQL的where多条件查询](http://blog.csdn.net/zbw18297786698/article/details/52904316)

    hash类型的条件查询，也不是严格意义的where

* [Redis教程](http://www.runoob.com/redis/redis-tutorial.html)

    我把它当做redis中文文档用
    
* [Redis Cluster](https://my.oschina.net/guol/blog/506193)
    
    redis Cluster集群配置，不用自己实现redis一致性hash算法
    
* [redis哨兵模式](http://wiki.jikexueyuan.com/project/redis/guard-mechanism.html)

    对redis哨兵配置以及原理分别进行了解释
    

## 解决工作中问题

* [从 XSL 参数中取值](https://msdn.microsoft.com/zh-cn/library/ms950787.aspx)

    微软msdn官方出品，把xsl中几乎所有参数情况都介绍完了，在解决xsl模板变量取值问题时帮了大忙

## github page  && Jekyll

* [如何创建免费博客 使用Github在线创建](https://jingyan.baidu.com/article/4853e1e5649f771909f72696.html)

    百度
    
* [创建GitHub技术博客全攻略](http://blog.csdn.net/renfufei/article/details/37725057/)
    
    CSDN
    
* [github 怎么搭建博客？](https://www.zhihu.com/question/23934523)
    
    知乎
    
* [傻瓜都可以利用github pages建博客](http://cyzus.github.io/2015/06/21/github-build-blog/)

    github
    
* [一步步在GitHub上创建博客主页](http://www.pchou.info/ssgithubPage/2013-01-03-build-github-blog-page-01.html)

    非常完整的教程
    
* [有哪些简洁明快的 Jekyll 模板？](https://www.zhihu.com/question/20223939)

    

* [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)
    
    阮一峰 

* [github + jekyll工作机制](http://www.360doc.com/content/14/0415/07/13232598_369075184.shtml)
    
    jekyll工作机制讲解很透彻

## 微信公众号

* [微信公众平台开发入门教程](http://www.cnblogs.com/txw1958/p/wechat-tutorial.html)

    方倍工作室出品，基于新浪云，新浪云其实比较坑，免费一个月，云豆烧不起，推荐亚马逊云，现在注册免费一年
    
* [CoderDream的博客](http://coderdream.iteye.com/category/325492)
* [柳峰微信公众号专栏](http://blog.csdn.net/lyq8479/article/category/1366622)

    关于微信公众号开发，这两个博客教程很全面

## git相关

* [Git及Github使用方法总结](http://blog.csdn.net/wangtaoking1/article/details/17115021)
    
    git常用命令总结
    
* [常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

    阮一峰的博客总结git常用命令，也是目前我通过git bash发布日志基本的常用命令
    
* [廖雪峰的git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

    初学git的时候看过一部分，介绍的很详细
    

## 优秀的博客

* [hawstein的博客](http://www.hawstein.com/)  
* [程序猿DD的博客](http://blog.didispace.com/)  
* [egg的博客](http://blog.csdn.net/zhangerqing/article/list/1)  
* [舞动的心](http://www.cnblogs.com/liuzhen1995/)  
* [王鸿飞的专栏](http://blog.csdn.net/neosmith)  
* [Defonds 的专栏](http://blog.csdn.net/defonds)

    
## 其它

* [免费的编程中文书籍索引](https://github.com/Hawstein/free-programming-books-zh_CN)

    非常全的一个书目列表，有生之年是涉猎不完了
    
* [网络上免费API接口](http://www.cnblogs.com/annie00/p/5852873.html)

    自己在做微信公众号需要调用第三方API时收录的，还是蛮全面的，由于本人的新浪云已欠费，公众号已停用，不过源码还在本人的[github](https://github.com/coldxiangyu/hairwechat)，实现了百度地图、百度翻译等API的调用
    
* [最好的网上学习网站](https://www.coursera.org/)

    自备英语功底

## Comments

{% include comments.html %}
