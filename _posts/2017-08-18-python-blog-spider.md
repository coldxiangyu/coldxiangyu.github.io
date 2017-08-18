---
layout: post
title:  "Python爬取我的博客图片"
date:   2017-08-18 13:36:22
author: coldxiangyu
categories: Python
tags: Python
mathjax: true
---

* content
{:toc}

## 前言

>人生苦短，我用Python

相信大部分程序员都看到过这句话，虽然传统的JAVA程序员大多没有学过Python，但这句话确实是深入人心。

当然，JAVA程序员不想学习Python也是有原因的，毕竟除去JAVA这门语言本身的学习成本，还有浩如烟海的整个JAVA生态。而JAVA研究尚未彻底，何谈再去学习另一门语言呢？

对于这种想法，我非常理解，然而却不赞同。原因参考我在今日头条的回答：[怎样才能更好地学习JAVA又快又好？](https://www.wukong.com/answer/6432187518795383042/)

归根结底，编程语言只是一个工具，用于人和计算机进行交流。你敲0和1，你用汇编，电脑很熟悉，理解起来很容易，都不需要多少解释。而你说的越简单，解释器给电脑解释起来就越麻烦。Python就是这样，编写过程十分舒服，解释器怎么费劲解释我不管，我自己爽了就好。而虽然Python速度不及很多语言，但又无伤大雅。

而JAVA，真的那么好用吗？答案是否定的，写了这么多年的JAVA，在那些条条框框里摸爬滚打，由于没有其他语言的对比，你永远都感受不到JAVA的笨重。至少，对比Python，JAVA就显得笨重多了。

当然，那些没有尝试过其他语言的人，也根本不会懂得，其实有了一门语言的基础，再扩展其他语言，只需要几个星期而已。

如果这样还不够，我打算再给你一个理由去学习Python：
参考知乎首答：[Python有怎样的玩法？](https://www.zhihu.com/question/20799742) 看完之后，你会觉得，原来学掌握一门语言可以做这么多事情。而我们学了JAVA却一直在维护大型的企业项目，开发基本的功能模块，不停的满足客户的需求。而我们真正的玩过一门语言吗？并没有。

当我们真的想用一门语言做一些有意思的事情的时候，Python或许是最优之选。它可大可小，最重要的，编程十分友好，简单快捷。

对于我目前而言，JAVA是用来吃饭的，而Python是用来玩的。

玩Python，爬虫永远是排在前面的课题。早在之前，我用java就写过爬虫，使用HttpClient和一些其他的框架，然而java确实不适合玩爬虫，代码量跟Python根本不是一个量级。

于是，经过这几个星期断断续续的学习Python的一些语法，我打算将我自己博客的所有图片爬取下来，作为一个学习成果的检验。

>备注：本人使用Python 3.5，由于实现功能并不复杂，也并未使用框架

## 实战

说下基本要求：将每篇博客的所有图片独立保存在以该博客名创建的文件夹下，父文件夹为该博客的发表日期，不同日期文件夹也相互独立。

要爬取所有博客的图片，首先就要了解一下博客页面的结构，来方便我们爬取链接。
首先打开我的第一篇博文，《Docker基本应用》，如何爬取整片的博客内容呢？方法如下：

```
def getContent(url):
    content = urllib.request.urlopen(url).read().decode('utf-8')
    return content
```
通过`urllib.request.urlopen`就可以轻松对url进行访问了。

接下来找到图片所对应的源码，如下图：

![image_1bnpl79ivmqp2ku1n8517ruvr08b.png-130.6kB][1]

正则匹配如下：

``` python
imagelist = re.findall('img src="(http.*?)"',content)
```

这只是一篇文章，要爬取所有文章就需要想办法了，一个就是通过每一篇博客下面的`上一篇`,`下一篇`的外链来进行持续查找，第二种办法就是通过我的博客另一个页面，展示的所有文章列表来获取，也就是下图页面：

![image_1bnpcg3b71m64f251vq81dvhv1m9.png-244.3kB][2]

再看看对应的源码：

![image_1bnpekese1i4lu753g31u4cnjgm.png-102.4kB][3]

正则匹配如下：
``` python
pattern = re.compile('<a class="title" href="(.*?)">.*?</a>',re.S)
linklist = re.findall(pattern,content)
```

可以看到，抓一个linklist出来也十分方便。

当然，这两种方式都可以。不过由于我抓取图片的时候本身就需要对每篇博文中的图片链接进行正则匹配，顺便也可以十分方便的获取到外链。因此，为了不增加额外的消耗，我选择第一种方式。但是如果博客页面中没有外链的话，就要考虑第二种方式了。

下面我们来看看外链的标签是怎样的，如下图：

![image_1bnnodpplun91it6jciu62ip1m.png-44.2kB][4]

正则匹配如下：
``` python
pattern = re.compile('<div class="pre">.*?<p>.*?<a href="(.*?)">.*?</a>',re.S)
linklist = re.findall(pattern,content)
```

我们可以看到一个RESTFUL风格的链接，加上我们的root_url，http://www.coldxiangyu.com，就可以组成一个完整的博客链接进行访问了。

由于我们没有采用第二种方式获取一个链接的list出来，也就无法根据链接进行循环处理，这时候我们可以采用递归调用的方式。

类似以下结构：
``` python
def call(url):
    # 获取外链
    # 拼接root_url为全新链接
    # call(new_url)
    pass
root_url = "http://www.coldxiangyu.com"
url = "http://www.coldxiangyu.com/2017/08/01/docker-intruduction/"
call(url)
```

接下来上源码:
``` python
# author: coldxiangyu
# encoding=utf-8

import re
import os
import urllib.request

def getContent(url):
    content = urllib.request.urlopen(url).read().decode('utf-8')
    return content

def getPreLink(content):
    pattern = re.compile(r'<head>.*?<title>(.*?)</title>.*?<link.*?canonical.*?href="(http.*?)">.*?</head>.*?<div class="pre">.*?<p>.*?<a href="(.*?)">.*?</a>.*?</p>.*?</div>.*?<div class="nex">', re.S)
    items = re.findall(pattern, content)
    return items[0]

def getImageList(content):
    imagelist = re.findall('img src="(http.*?)"',content)
    return imagelist

def call(url):
    content = getContent(url)
    item = getPreLink(content)
    imagelist = getImageList(content)
    print('《' + item[0] + '》','共有', len(imagelist), '张图片需要保存！')
    if len(imagelist) > 0:
        saveImage(item, imagelist)
        print('保存完毕：' + url)
    else:
        print('没有可保存的图片，跳过！')
    try:
        if item[2] != '':
            new_url = root_url + item[2]
            call(new_url)
    except Exception as e:
        print("打印完毕！");

def saveImage(item, imagelist):
    arrpath = item[1].split('/')
    filepath = arrpath[3] + arrpath[4]
    path = 'D:/MyDownload/blog/' + filepath + '/' + item[0] + '/'
    if not os.path.exists(path):
        os.makedirs(path)
    count = 0
    for url in imagelist:
        print(url)
        if(url.find('.') != -1):
            name = url[url.find('.', len(url) - 5):];
            bytes = urllib.request.urlopen(url)
            f = open(path + str(count) + name, 'wb')
            f.write(bytes.read())
            f.flush()
            f.close()
            count += 1


root_url = "http://www.coldxiangyu.com"
url = "http://www.coldxiangyu.com/2017/08/01/docker-intruduction/"
call(url)

```
实现过程并不复杂，相信只要你有任何一门语言的基础，都能看懂。

后台日志部分截图如下：

![image_1bnpkvpa71liosr61d6c15r6qfv5n.png-132.2kB][5]

最终效果如下：

![image_1bnpl1aicipd1ek695g5nh1it66k.png-34kB][6]

![image_1bnpl1v4n1hc61kqstic79s62k7h.png-31.6kB][7]

![image_1bnpl3fl61kvogq81kq017jf1rht7u.png-80.7kB][8]

当然，代码里有许多不完善的地方，只是保证功能得以实现，关于Python的一些高级应用以及框架，会在以后进行说明展示。
<br/><br/>

>版权声明：本文为原创内容，转载请注明出处，谢谢合作。

  [1]: http://static.zybuluo.com/coldxiangyu/xgbx6fem8yb8bpxvpipeiroc/image_1bnpl79ivmqp2ku1n8517ruvr08b.png
  [2]: http://static.zybuluo.com/coldxiangyu/cqqdkr5ybrqy4mbs6fv4rlkn/image_1bnpcg3b71m64f251vq81dvhv1m9.png
  [3]: http://static.zybuluo.com/coldxiangyu/9ek3vdt6i4oeyu052lilb9ye/image_1bnpekese1i4lu753g31u4cnjgm.png
  [4]: http://static.zybuluo.com/coldxiangyu/knrywwmrq9mu3a4sta0u422v/image_1bnnodpplun91it6jciu62ip1m.png
  [5]: http://static.zybuluo.com/coldxiangyu/g4ijjo47enhyg6k6ezq59gpv/image_1bnpkvpa71liosr61d6c15r6qfv5n.png
  [6]: http://static.zybuluo.com/coldxiangyu/wa79z6fsxy1zfak1mgqyp5d4/image_1bnpl1aicipd1ek695g5nh1it66k.png
  [7]: http://static.zybuluo.com/coldxiangyu/u5pu9ur3dn459ids488lsw04/image_1bnpl1v4n1hc61kqstic79s62k7h.png
  [8]: http://static.zybuluo.com/coldxiangyu/3h0xawbr0pedugam6vtrkr3x/image_1bnpl3fl61kvogq81kq017jf1rht7u.png

