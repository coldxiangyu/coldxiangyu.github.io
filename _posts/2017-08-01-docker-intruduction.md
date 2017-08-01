# Docker基本应用

标签（空格分隔）： Docker

---

## Docker杂谈
最近在研究Docker。
Docker火吗？试着去了解一下它的发展史就会发现，Docker是诞生于2013年的开源项目，算是我们接触到的非常年轻的明星项目了。在这期间，不管是一线大佬还是普通程序员，对Docker的关注和讨论度都是空前的。人们对Docker这种打破传统思维的工作方式趋之若骛，大有打破传统模式一统天下之势。
但是，当我们不随流，并且静下心来想一想，Docker其实并没有想象中那么火。关于Docker，大部分人只是盲目追求这个ID，论坛上大多议论的话题就是：Docker为什么这么火？超过半数的人只是知道这个概念，并没有对此进行学习研究。而对于Docker敢于尝鲜没有包袱的也大多是小型的互联网公司，他们坚定不移的拥抱开源，有着大无畏的互联网精神。还有一部分人虽有余力对Docker进行研究，而在实际工作中，再三评估，不敢贸然使用。以至于，除了目前几个国内的巨头，确实没有什么拿得出手的成熟的Docker产品。而且Docker一路走过来，也难说稳定。由于这些原因，到现在Docker基本上也是各家玩各家的，因此，‘天下大同’的美好构想还有很长的路要走。毕竟目前整个行业并没有接受以容器镜像为主要形态的软件发布模式，况且，分布式以及微服务在目前的行业中也算不上普及。
Docker还能火多久，谁知道呢？也许过几年就有其他的替代品了，但是Docker给我们带来的全新的思想，将会一直影响我们。这，也是我们要学习它的原因。
## Docker概述
本着原创的精神，我在这里尽量不搬弄那些已经被大家说烂了的概念，只谈谈自己对Docker的理解。
Docker的历史还蛮有意思的，有兴趣的可以去自己找相关资料看一下，我从Docker的历史中给你们划一个重点，就是开源精神。可以说，没有开源，也就没有现在的Docker，它很可能依旧还是那个默默无闻的dotCloud。
Docker翻译过来是集装箱的意思，当然，Docker的图标也是一个很好的体现。其实把Docker比作集装箱并不是很恰当的，它应该是装集装箱的那个货船抑或管理集装箱的码头。虽然说了不搞概念，但是关于Docker，有几个概念不得不说，因为Docker的核心就在于此。它们分别是：镜像，容器和仓库。
关于这几个的概念，我看到有人把镜像比作类，容器比作对象。我觉得比较牵强，强行转换概念很容易让大家对这些东西更陌生。下面我来说一下我的认识：
简单来说，每一个镜像相当于一个Linux+software，它是一个分层的文件系统，底层由Linux系统支撑，上层添加应用。
比如我们部署Linux服务器的时候，要添加tomcat应用。在Docker中，这就是一个tomcat的镜像，当你启动这个镜像，它就是一个容器了。而实际上，要搞清楚镜像和容器本质，要复杂的多。这里不进行详细解释。
关于仓库，大家都不算陌生了，比如maven仓库，git仓库我们也都接触的不少了。Docker仓库一般指的是Docker Hub，提供大量的镜像。
这篇文章本身并不是要介绍这些概念的，我已经预感到这篇文章会比较长，而我又不想再另写一篇，因此一些比较基本的东西就不再讲了。

## Docker环境搭建
### Linux安装
Docker需要在Linux环境运行，且Linux内核版本不能低于3.10，由于Docker是在Ubuntu环境下进行开发的，因此建议使用Ubuntu系统。
我本地虚拟机之前安装过CentOS6.5，但是内核版本只有2.*，需要升级内核，相对麻烦。然后我果断放弃CentOS，使用Ubuntu一步到位。我们在研究其他技术也是一样，尽量一步到位，省去了那些研究之外遇到的一些莫名其妙的麻烦。
虽然Docker有Windows安装版，但实质也是在windows运行虚拟机，不推荐。

我使用的Ubuntu版本是14.04.1，内核版本4.4.0：
```
root@coldxiangyu-virtual-machine:/bin# uname -a
Linux coldxiangyu-virtual-machine 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:07:32 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```
VMware安装虚拟机我就不再说明了，合格的程序员必备技能。
虚拟机安装完成之后，尽量将默认源替换为国内源，以提高下载速度以及稳定性。如果你使用默认源没有问题，就不用替换了。
我使用的阿里源，速度还可以，你也可以替换为搜狐的或着其他源。
路径/etc/apt/source.list
```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb [arch=amd64] https://download.docker.com/linux/ubuntu trusty edge
# deb-src [arch=amd64] https://download.docker.com/linux/ubuntu trusty edge

```
改完之后运行命令：`sudo apt-get update`
其他类似vim，VMware Tools自己看着装就好了，没有vim我是不习惯的。
如果你想通过SecureCRT连接虚拟机的话，记住我下面这张图，应该会帮到你。
![image_1bm1lc3r211gh1bj51u5l1khda1f9.png-42.8kB][1]
此时root用户一般是没有密码设置的，通过`sudo passwd`进行root用户密码设置，接下来我们会用到root权限。

### 安装Docker
此时，万事俱备，我们可以安装Docker了。
通过`wget -qO- https://get.docker.com/ | sh`下载最新Docker版本。
我们可以使用`docker`或者`docker command --help`查看docker客户端命令：
![image_1bmbt0oh916pc1hln1ndv1vpa18rr9.png-106.6kB][2]
这时候，Docker我们就安装成功了。

## Docker容器
### Hello world
使用`sudo service docker start`命令启动docker后台服务：
```
root@coldxiangyu-virtual-machine:~# sudo service docker start
start: Job is already running: docker
```

当然，Docker也有自己的hello world：
```
root@coldxiangyu-virtual-machine:~# docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world

```
### 容器交互
其中ubuntu:15.10指定要运行的镜像，Docker首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
那么，我们如何进入容器内部进行操作呢？
我们通过-i -t命令即可与容器进行交互：
```
root@coldxiangyu-virtual-machine:~# docker run -i -t ubuntu:15.10 /bin/bash
root@a8068d88ad1f:/#
```
其中-i:允许你对容器内的标准输入 (STDIN)进行交互,-t:在新容器内指定一个伪终端或终端。
容器内文件结构如下：
![2017-07-31 17:07:24屏幕截图.png-25.5kB][3]
我们可以通过运行exit命令或者使用CTRL+D来退出容器。
### 后台运行容器
那么我们如何以后台运行的形式来运行容器呢？
我们通过容器启动一个脚本：
```
root@coldxiangyu-virtual-machine:~# docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo this is coldxiangyu; sleep 1; done"
4a4c28123edd393d082d8a7b4c1d587520593d3f0f8abc673d18f6816795ad70
```
可以看到我们得到了一个ID号，这个就是容器ID了。我们通过`docker ps`可以来查看运行的Docker容器。
![2017-07-31 17:15:02屏幕截图.png-29.5kB][4]
这时候，我们可以看到系统还默认给容器生成了一个NAMES：vigilant_ptolemy
我们可以通过`docker logs`命令来查看容器内输出日志，使用ID和NAME均可：
![2017-07-31 17:18:10屏幕截图.png-51.7kB][5]
接下来我们使用`docker stop`来停止这个容器：
![2017-07-31 17:20:36屏幕截图.png-28.1kB][6]
整个过程是非常简单的。

### web容器实例
以上的例子只是做一些基本的输出，与实际web部署是不符的。我们拿个现成的web例子来运行一下：
```
runoob@runoob:~# docker run -d -P training/webapp python app.py
```
其中-d:让容器在后台运行，-P:将容器内部使用的网络端口映射到我们使用的主机上，Docker会自动分配本地映射端口。
![2017-08-01 09:21:42屏幕截图.png-39.1kB][7]
可以看到，这时候多了PORTS端口映射信息。意思是Docker将应用的原端口（5000）映射到本机的32768端口。我们本地访问一下：
![2017-08-01 09:23:30屏幕截图.png-8kB][8]
查看日志：
![2017-08-01 09:24:44屏幕截图.png-39.8kB][9]
当然，如果你不想使用Docker自动分配的端口，也可以自己通过`-p`命令指定本地映射端口：
`docker run -d -p 10086:5000 training/webapp python app.py`
这里一定要注意本地端口和原端口的顺序，本地端口：docker应用端口，如下：
![2017-08-01 09:40:02屏幕截图.png-37.1kB][10]
可以看到，我们现在同时运行了两个容器，其中一个映射本地端口：32768，另外一个映射本地端口：10086。此时我们通过浏览器访问本地10086端口：
![2017-08-01 09:42:30屏幕截图.png-8.1kB][11]
当我们启动的容器比较多，这时候通过`docker ps`命令列出的容器很多，不方便我们查看，我们仅仅需要列出最后一次创建的容器，这时候我们使用`docker ps -l`即可。
![2017-08-01 09:56:16屏幕截图.png-19kB][12]
### 其它命令
我们在Linux上经常通过`top`命令来查看系统信息，包括进程以及CUP以及内存占用等等，Docker也有一个类似的命令，我们可以通过`docker top`来查看容器进程：
![2017-08-01 09:58:00屏幕截图.png-31.6kB][13]
此外，我们可以通过`docker start [CONTAINER]`来启动被`docker stop`的容器。也可以通过`docker restart`来重启仍在运行的容器。如果要删除一个容器的话，通过`docker rm [CONTAINER]`即可实现。
每一个容器的启动都是由镜像中json配置文件来加载配置的，我们可以通过`docker inspect`来进行查看：
![2017-08-01 10:02:31屏幕截图.png-44.9kB][14]
关于docker容器的使用命令，先介绍这么多。

怎么样，有没有体会到Docker给我们带来的便利呢？

## Docker镜像 
介绍完了容器，我们来介绍一下镜像。
我们已经知道，容器就是一个镜像的运行实例。镜像是静态的，容器是动态的。我大概说一下镜像转化为容器的大概过程：进程肯定不能无中生有，这个转化工作肯定要由docker守护进程来执行，docker守护进程通过加载docker镜像的json配置，为容器初始化环境变量，并运行docker镜像指定的进程，完成容器的启动。
使用docker，我们可以定义自己的容器并创建镜像并上传。整个过程对于我们来说并不陌生，跟通过git操作远程仓库是一样的。

### 查看本地镜像
我们可以通过`docker images`开查看本地镜像：
![2017-08-01 10:54:20屏幕截图.png-29.9kB][15]

其中，

- REPOSTITORY：表示镜像的仓库源
- TAG：镜像的标签
- IMAGE ID：镜像ID
- CREATED：镜像创建时间
- SIZE：镜像大小

一个`REPOSTITORY:TAG`可以唯一标识一个镜像，同一个REPOSTITORY可以有多个TAG，也就是版本。
在上面我说过运行ubuntu的例子中，`docker run -t -i ubuntu:15.10 /bin/bash`，如果不指定TAG，只使用ubuntu，docker会默认加载ubuntu:latest，也就是最近一版的镜像。

### 查找镜像

如果我们想查找镜像的话，可以通过https://hub.docker.com/也就是Docker Hub去搜索镜像，也可以通过命令`docker search`直接查找，比如我们要找tomcat相关镜像：
![2017-08-01 11:07:17屏幕截图.png-113.1kB][16]
可以看到有许多搜索结果，其中OFFICIAL是Docker官方发布镜像。

### 下载镜像

我们通过`docker pull`来下载docker发布的最新的官方镜像tomcat：
![2017-08-01 11:12:03屏幕截图.png-55.9kB][17]
通过下载界面，我们再次印证了镜像是多层的这个概念。
下载完成后，我们就可以通过`docker run tomcat`来启动这个镜像了。
![2017-08-01 11:59:31屏幕截图.png-132.6kB][18]

### 创建镜像

如果从仓库中下载的镜像不能满足我们的需求，我们也可以自己创建自己定制的镜像。那么如何创建镜像呢？方法有二：
1.通过容器创建
2.通过Dockerfile进行创建

#### 通过容器创建
以我们本地下载的ubuntu:15.10为例，我们将此镜像启动为一个容器。
通过`docker run -t -i ubuntu:15.10 /bin/bash`命令进入此容器，可以看到容器ID为`338f23cb15dd`，执行apt-get update 更新。
![2017-08-01 12:05:34屏幕截图.png-142.4kB][19]
通过Ctrl+D或exit退出此容器，并将此容器提交为一个新镜像。
```
root@coldxiangyu-virtual-machine:/home/coldxiangyu# docker commit -m="update as v2" -a="coldxiangyu" 338f23cb15dd coldxiangyu/ubuntu:v2
sha256:3990f72176c0af1c2bc269b65124a4923ba5e51957f79985cf8fc265300f7bff
```
其中`docker commit -m`与`git commit -m`是一样的，都是提交信息，-a为提交作者，后面跟容器ID，加上REPOSTITORY:TAG目标镜像。
这时候我们再通过`docker image`查看本地镜像，可以看到`coldxiangyu/ubuntu:v2`镜像已经存在。
![2017-08-01 12:14:17屏幕截图.png-50.7kB][20]
当我们想使用新建的这个镜像启动容器的时候，通过`docker run -t -i coldxiangyu/ubuntu:v2 /bin/bash`直接启动即可。

#### 通过Dockerfile创建
由以上通过容器创建镜像是非常麻烦的，你需要在容器shell中把所有的安装都进行一遍，这太麻烦了。我们可以不依赖容器，自己通过Dockerfile构建一个镜像。其实也就是在把命令放到统一的Dockerfile执行，我们来看一下Dockerfile大致的构造：
![2017-08-01 14:03:34屏幕截图.png-32.4kB][21]
首先，每一个指令都会在镜像中创建一个新的层，每个指令需要大写。
`FROM`指令，作为Dockerfile中的第一条指令，为接下来的指令提供镜像基础。
`MAINTAINER`声明维护者和联系方式。
`RUN`指令就是在镜像中执行shell命令。
`CMD`表示镜像运行默认参数，可存在多个，可被覆盖，只有最后一个生效。
`EXPOSE`指令设置镜像对外暴露端口。
`ENTRYPOINT/CMD`，一般两者可以配合使用，比如：
```
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]
```
在docker daemon模式下，无论你是使用ENTRYPOINT，还是CMD，最后的命令，一定要是当前进程需要一直运行的，才能够防容器退出。
执行`docker build -t coldxiangyu/ubuntu:14.04 .`命令，运行我们创建的Dockerfile：
```
root@coldxiangyu-virtual-machine:/home/coldxiangyu# docker build -t coldxiangyu/ubuntu:14.04 .
Sending build context to Docker daemon  627.5MB
Step 1/9 : FROM ubuntu:14.04
14.04: Pulling from library/ubuntu
7ee37f181318: Pull complete 
df5ffabe5e97: Pull complete 
ae2040ed51a1: Pull complete 
3ce7010d244b: Pull complete 
2538b201d2a6: Pull complete 
Digest: sha256:13eecbc0e57928c1cb3ccca3581e2a6f4b0f39c1acf5a279e740e478accd119b
Status: Downloaded newer image for ubuntu:14.04
 ---> 54333f1de4ed
Step 2/9 : MAINTAINER coldxiangyu	"coldxiangyu@qq.com"
 ---> Running in 644cc9afc189
 ---> 8565798e43f2
Removing intermediate container 644cc9afc189
Step 3/9 : RUN /bin/echo 'root:123456' |chpasswd
 ---> Running in 0a76a1ed781d
 ---> 56c42008bca9
Removing intermediate container 0a76a1ed781d
Step 4/9 : RUN useradd coldxiangyu
 ---> Running in 9443a7d49388
 ---> 1eefc11dc6ff
Removing intermediate container 9443a7d49388
Step 5/9 : RUN /bin/echo 'coldxiangyu:123456' |chpasswd
 ---> Running in 5e0983be45df
 ---> fce39410e0c2
Removing intermediate container 5e0983be45df
Step 6/9 : RUN /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
 ---> Running in 453b15056392
 ---> 2d69553fb772
Removing intermediate container 453b15056392
Step 7/9 : EXPOSE 22
 ---> Running in caf583e7d745
 ---> 993cf96579f6
Removing intermediate container caf583e7d745
Step 8/9 : EXPOSE 80
 ---> Running in de1ecb8de78f
 ---> f5bfec891b2c
Removing intermediate container de1ecb8de78f
Step 9/9 : CMD /usr/sbin/sshd	-D
 ---> Running in 301d6b359403
 ---> 5600f326e2c9
Removing intermediate container 301d6b359403
Successfully built 5600f326e2c9
Successfully tagged coldxiangyu/ubuntu:14.04
root@coldxiangyu-virtual-machine:/home/coldxiangyu#
```
我们可以看到通过Dockerfile创建镜像的完整过程。
此时我们再查看本地镜像列表：
![2017-08-01 14:21:19屏幕截图.png-46.2kB][22]
可以看到coldxiangyu/ubuntu:14.04镜像已经创建成功。
在这里我再提一下我之前说的镜像分层，以及Dockerfile每一个指令都会在镜像中创建新的一层。虽然我们在镜像列表对用户展示的只有一层，而实际上，你看到的只是最上面一层，镜像的每层是层层覆盖的，每一层和每一层都是关联的。就像《三体》里面的二维化一样，将多层压缩为一个面，也像是一个俯视图。
我们通过`docker history`命令来验证一下我的说法：
![2017-08-01 14:29:11屏幕截图.png-94.3kB][23]
这时候，我们能看到镜像的每一个分层对应的Dockerfile中的创建指令。
如果你足够聪明的话，你会注意我上面还说了一句：
>虽然我们在镜像列表对用户展示的只有一层，而实际上，你看到的只是最上面一层

这时候你看一下历史版本的最上层镜像ID，再对比一下`docker images`镜像列表中展示的镜像ID，他们是一样的。

## 总结
到这里，Docker的基本应用就介绍到这里，剩下的就由大家自己去摸索了。
最后首尾呼应一下，Docker为什么这么火，你也许已经有了自己的理解。


  [1]: http://static.zybuluo.com/coldxiangyu/k2x6gypqt6qa9410olo099ke/image_1bm1lc3r211gh1bj51u5l1khda1f9.png
  [2]: http://static.zybuluo.com/coldxiangyu/ap2ddd4k8p8obqx4f3y8yd8q/image_1bmbt0oh916pc1hln1ndv1vpa18rr9.png
  [3]: http://static.zybuluo.com/coldxiangyu/3vasd6pqhptcx5gfq5adoy12/2017-07-31%2017:07:24%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [4]: http://static.zybuluo.com/coldxiangyu/gbe81ljoulcj2zs18gggk0o0/2017-07-31%2017:15:02%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [5]: http://static.zybuluo.com/coldxiangyu/42niwy0vax0i5w9jpyog123b/2017-07-31%2017:18:10%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [6]: http://static.zybuluo.com/coldxiangyu/3rc8yt4qf17mvd9vef4ce2m5/2017-07-31%2017:20:36%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [7]: http://static.zybuluo.com/coldxiangyu/qqr12nwwl6m631hjf0g0lv84/2017-08-01%2009:21:42%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [8]: http://static.zybuluo.com/coldxiangyu/g18kr266a4itvkjb847pdzqq/2017-08-01%2009:23:30%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [9]: http://static.zybuluo.com/coldxiangyu/o3k5pk2fmblbwtfcvdu7o8g6/2017-08-01%2009:24:44%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [10]: http://static.zybuluo.com/coldxiangyu/ej799wlbz174ejxyid4ufl09/2017-08-01%2009:40:02%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [11]: http://static.zybuluo.com/coldxiangyu/dv7kob33vxm10cu21fvu7fvk/2017-08-01%2009:42:30%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [12]: http://static.zybuluo.com/coldxiangyu/uualcuk7lfxo40i4rkvs06al/2017-08-01%2009:56:16%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [13]: http://static.zybuluo.com/coldxiangyu/ap2gy3eoh0p5rn51mysb7jmj/2017-08-01%2009:58:00%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [14]: http://static.zybuluo.com/coldxiangyu/c0tvdo0e0jehuoeami4jztkq/2017-08-01%2010:02:31%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [15]: http://static.zybuluo.com/coldxiangyu/nohc0q5fw4j7wg285cctet4u/2017-08-01%2010:54:20%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [16]: http://static.zybuluo.com/coldxiangyu/azdp8t3i7cjdud91uveqng2n/2017-08-01%2011:07:17%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [17]: http://static.zybuluo.com/coldxiangyu/251tkabuvjxbpjirl4cb727a/2017-08-01%2011:12:03%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [18]: http://static.zybuluo.com/coldxiangyu/bohuxs3fzay1slau21peisy3/2017-08-01%2011:59:31%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [19]: http://static.zybuluo.com/coldxiangyu/7i0ti1wq8eswf8ju1llm2mbj/2017-08-01%2012:05:34%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [20]: http://static.zybuluo.com/coldxiangyu/w9wgze5zxzq370yu84d4nxyu/2017-08-01%2012:14:17%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [21]: http://static.zybuluo.com/coldxiangyu/go33ub360jyrc1683cgxdrzf/2017-08-01%2014:03:34%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [22]: http://static.zybuluo.com/coldxiangyu/shcpcgj8xo6p50t6cyhpyego/2017-08-01%2014:21:19%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png
  [23]: http://static.zybuluo.com/coldxiangyu/pnlc3sgrwzway4vkzq33jciz/2017-08-01%2014:29:11%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png