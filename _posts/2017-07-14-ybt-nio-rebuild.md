---
layout: post
title:  "银保通Socket改造NIO"
date:   2017-07-14 13:36:22
author: coldxiangyu
categories: java
tags: NIO Socket
mathjax: true
---

* content
{:toc}


## 前言
在[对银保通的深度思考](http://www.coldxiangyu.com/2017/07/10/%E5%AF%B9%E9%93%B6%E4%BF%9D%E9%80%9A%E7%9A%84%E6%B7%B1%E5%BA%A6%E6%80%9D%E8%80%83/)文章中我已经说明了，目前银保通采用阻塞Socket，对线程资源浪费严重，针对此，我考虑对它进行NIO系统改造。
其实NIO早在JDK1.4就已经发布，并不是什么新技术，银保通虽然比较老，但本身也是基于JDK1.5搭建的。在最初的架构设计中，没有采用非阻塞的NIO或者异步AIO，算是一个遗憾吧。  

今天我们将在银保通现有架构的基础上进行NIO改造，对于NIO的原理以及API还不熟悉的同学需要对NIO有一个整体的理解再来看代码，我尽可能的将NIO代码进行详细注释。  




## SHOW THE CODE

### ServerHandle.java

``` java
package com.sinosoft.midplat.net;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

import org.apache.log4j.Logger;
import org.jdom.Element;
  
public class ServerHandle implements Runnable{  
    private int ttPort = -1;
    private Selector selector;  
    private ServerSocketChannel serverChannel;  
    private volatile boolean started;
    private final static Logger cLogger = Logger.getLogger(ServerHandle.class);
    
    /** 
     * 构造方法 
     * @param Element Socket 传入socket节点 
     */  
	public ServerHandle(Element ttSocket) throws Exception {
    	ttPort = Integer.parseInt(ttSocket.getChildText("port"));
        try{  
            //创建选择器  
            selector = Selector.open();  
            //打开监听通道  
            serverChannel = ServerSocketChannel.open();  
            //如果为 true，则此通道将被置于阻塞模式；如果为 false，则此通道将被置于非阻塞模式  
            serverChannel.configureBlocking(false);//开启非阻塞模式  
            //绑定端口 backlog设为1024  
            serverChannel.socket().bind(new InetSocketAddress(ttPort),1024);
            //监听客户端连接请求  
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);
            cLogger.info("监听端口：" + ttPort);
            cLogger.info("Socket(" + ttSocket.getChildText("name") + ")加载成功!");
            //标记服务器已开启  
            started = true;  
        }catch(IOException e){  
        	cLogger.error("创建服务器套接字失败！" + ttPort, e);
			return; 
        }  
    }  
    public void stop(){  
        started = false;  
    }  
    @Override  
    public void run() {  
        //循环遍历selector  
        while(started){  
            try{  
                //无论是否有读写事件发生，selector每隔1s被唤醒一次  
                selector.select(1000);  
                //阻塞,只有当至少一个注册的事件发生的时候才会继续.  
//              selector.select();  
                Set<SelectionKey> keys = selector.selectedKeys();  
                Iterator<SelectionKey> it = keys.iterator();  
                SelectionKey key = null;  
                while(it.hasNext()){  
                    key = it.next();  
                    it.remove();  
                    try{  
                        handleInput(key);  
                    }catch(Exception e){  
                        if(key != null){  
                            key.cancel();  
                            if(key.channel() != null){  
                                key.channel().close();  
                            }  
                        }  
                    }  
                }  
            }catch(Throwable t){  
                t.printStackTrace();  
            }  
        }  
        //selector关闭后会自动释放里面管理的资源  
        if(selector != null)  
            try{  
                selector.close();  
            }catch (Exception e) {  
                e.printStackTrace();  
            }  
    }
    
    private void handleInput(SelectionKey key) throws IOException{  
        if(key.isValid()){  
            //处理新接入的请求消息  
            if(key.isAcceptable()){  
                ServerSocketChannel ssc = (ServerSocketChannel) key.channel();  
                //通过ServerSocketChannel的accept创建SocketChannel实例  
                //完成该操作意味着完成TCP三次握手，TCP物理链路正式建立  
                SocketChannel sc = ssc.accept();  
                //设置为非阻塞的  
                sc.configureBlocking(false);  
                //注册为读  
                sc.register(selector, SelectionKey.OP_READ);  
            }  
            //读消息  
            if(key.isReadable()){  
                SocketChannel sc = (SocketChannel) key.channel();  
                //创建ByteBuffer，并开辟一个1M的缓冲区  
                ByteBuffer buffer = ByteBuffer.allocate(1024);  
                //读取请求码流，返回读取到的字节数  
                int readBytes = sc.read(buffer);  
                //读取到字节，对字节进行编解码  
                if(readBytes>0){  
                    //将缓冲区当前的limit设置为position=0，用于后续对缓冲区的读取操作  
                    buffer.flip();  
                    //根据缓冲区可读字节数创建字节数组  
                    byte[] bytes = new byte[buffer.remaining()];  
                    //将缓冲区可读字节数组复制到新建的数组中  
                    buffer.get(bytes);  
                    String expression = new String(bytes);  
                    System.out.println(ttPort + " -- 服务器收到消息：" + expression);  
                    //处理数据  
                    String result = null;  
                    try{  
                        result = expression;  
                    }catch(Exception e){  
                        result = "返回结果处理异常：" + e.getMessage();  
                    }  
                    //发送应答消息  
                    doWrite(sc,result);  
                }  
                //没有读取到字节 忽略  
//              else if(readBytes==0);  
                //链路已经关闭，释放资源  
                else if(readBytes<0){  
                    key.cancel();  
                    sc.close();  
                }  
            }  
        }  
    }  
    //异步发送应答消息  
    private void doWrite(SocketChannel channel,String response) throws IOException{  
        //将消息编码为字节数组  
        byte[] bytes = response.getBytes();  
        //根据数组容量创建ByteBuffer  
        ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);  
        //将字节数组复制到缓冲区  
        writeBuffer.put(bytes);  
        //flip操作  
        writeBuffer.flip();  
        //发送缓冲区的字节数组  
        channel.write(writeBuffer);  
    }  
}  

```
### ServerListener.java  

``` java
package com.sinosoft.midplat.net;

import java.util.List;

import org.jdom.Document;
import org.jdom.Element;

public class ServerListener {  
    private static ServerHandle serverHandle;
    
    @SuppressWarnings("unchecked")
	public static void start() throws Exception{  
    	Document mSocketConfDoc = SocketConf.newInstance().getConf();
		List<Element> mSocketList = mSocketConfDoc.getRootElement().getChildren("socket");
		int mSize = mSocketList.size();
		for (int i = 0; i < mSize; i++) {
			Element ttSocket = mSocketList.get(i);
			start(ttSocket);  
		}
    }  
    public static synchronized void start(Element ttSocket) throws Exception{  
        serverHandle = new ServerHandle(ttSocket);  
        new Thread(serverHandle,"Server").start();
    }  
    public static void main(String[] args) throws Exception{
        start();  
    }  
}  
```

这样，我们整个服务端就已经写好了。为了方便大家理解，在这里描述一下整体思路：  
1. 通过ServerListener类读取socket.xml信息，遍历每个监听端口并通过ServerHandle类创建一个Reactor线程，将每一个ServerSocketChannel注册到该线程的Selector上监听ACCEPT事件。  
2. Selector轮询准备就绪的key，当Selector监听到新的客户端接入，处理新的接入请求，完成TCP三次握手，建立物理链路。  
3. 设置客户端链路为非阻塞模式，将新接入的客户端连接注册到Reactor线程的Selector上，监听读操作，读取客户端发送的网络消息，异步读取客户端消息到缓冲区。  
4. 将应答消息编码为Buffer，调用SocketChannel的write将消息异步发送给客户端。  

>说明：银保通的socket端口以及业务处理线程类是通过socket.xml配置的，大家在如果在本地试运行的话，需要将读取xml节点的部分进行改造

>由于银保通工程是GBK编码的，客户端发送报文也是GBK编码，因此在程序中并未对编码进行处理，大家在实际应用中，要对编码单独处理。

>通过程序可以看出，我在读取客户端信息之后原封不动的进行客户端返回了，并没有进行逻辑处理，此处在实际应用过程中要接入实际业务，在银保通就是实例化配置的线程业务类进行处理

OK，到这里我们启动服务端进行测试：  
```
13:58:01,916 INFO net.SocketConf(27) - Into SocketConf.load()...
13:58:01,920 DEBUG common.SysInfo(23) - basepath.r = /D:/workspace/AB_LIFE/ui/WEB-INF/classes/basepath.r
13:58:01,921 DEBUG common.SysInfo(25) - BasePath = /D:/workspace/AB_LIFE/ui/WEB-INF/classes/
13:58:01,922 DEBUG common.SysInfo(31) - MIDPLAT_HOME = null
13:58:01,923 DEBUG common.SysInfo(47) - Home = /D:/workspace/AB_LIFE/ui/WEB-INF/
13:58:01,923 INFO net.SocketConf(30) - Start load /D:/workspace/AB_LIFE/ui/WEB-INF/conf/socket.xml...
13:58:01,925 INFO common.XmlConf(52) - conf file modified at (2017-07-11 14:26:57,427) and length=6623 bytes!
13:58:02,097 INFO net.SocketConf(46) - End load /D:/workspace/AB_LIFE/ui/WEB-INF/conf/socket.xml!
13:58:02,098 INFO midplat.MidplatConf(32) - Into MidplatConf.load()...
13:58:02,099 INFO midplat.MidplatConf(35) - Start load /D:/workspace/AB_LIFE/ui/WEB-INF/conf/midplat.xml...
13:58:02,099 INFO common.XmlConf(52) - conf file modified at (2017-06-13 17:54:36,275) and length=11191 bytes!
13:58:02,117 INFO midplat.MidplatConf(51) - End load /D:/workspace/AB_LIFE/ui/WEB-INF/conf/midplat.xml!
13:58:02,122 INFO midplat.MidplatConf(91) - Out MidplatConf.load()!
13:58:02,122 INFO net.SocketConf(60) - Out SocketConf.load()!
13:58:02,152 INFO net.ServerHandle(40) - 监听端口：50100
13:58:02,152 INFO net.ServerHandle(41) - Socket(工行)加载成功!
13:58:02,154 INFO net.ServerHandle(40) - 监听端口：50101
13:58:02,154 INFO net.ServerHandle(41) - Socket(浙江工行)加载成功!
13:58:02,156 INFO net.ServerHandle(40) - 监听端口：50300
13:58:02,156 INFO net.ServerHandle(41) - Socket(建行)加载成功!
13:58:02,158 INFO net.ServerHandle(40) - 监听端口：50400
13:58:02,158 INFO net.ServerHandle(41) - Socket(农行)加载成功!
13:58:02,160 INFO net.ServerHandle(40) - 监听端口：60100
13:58:02,160 INFO net.ServerHandle(41) - Socket(成都农商银行)加载成功!
13:58:02,161 INFO net.ServerHandle(40) - 监听端口：60200
13:58:02,162 INFO net.ServerHandle(41) - Socket(北京农商银行)加载成功!
13:58:02,163 INFO net.ServerHandle(40) - 监听端口：50600
13:58:02,163 INFO net.ServerHandle(41) - Socket(邮政储蓄银行)加载成功!
13:58:02,164 INFO net.ServerHandle(40) - 监听端口：50180
13:58:02,165 INFO net.ServerHandle(41) - Socket(光大银行)加载成功!
13:58:02,166 INFO net.ServerHandle(40) - 监听端口：50190
13:58:02,166 INFO net.ServerHandle(41) - Socket(招行)加载成功!
13:58:02,168 INFO net.ServerHandle(40) - 监听端口：50200
13:58:02,168 INFO net.ServerHandle(41) - Socket(中信银行)加载成功!
13:58:02,170 INFO net.ServerHandle(40) - 监听端口：50201
13:58:02,170 INFO net.ServerHandle(41) - Socket(中信银行杭州保单贷款)加载成功!
13:58:02,172 INFO net.ServerHandle(40) - 监听端口：50210
13:58:02,172 INFO net.ServerHandle(41) - Socket(交通银行)加载成功!
13:58:02,174 INFO net.ServerHandle(40) - 监听端口：50220
13:58:02,174 INFO net.ServerHandle(41) - Socket(华夏银行)加载成功!
13:58:02,175 INFO net.ServerHandle(40) - 监听端口：60400
13:58:02,175 INFO net.ServerHandle(41) - Socket(辽宁农信社)加载成功!
13:58:02,177 INFO net.ServerHandle(40) - 监听端口：60500
13:58:02,177 INFO net.ServerHandle(41) - Socket(黑龙江农信社)加载成功!
13:58:02,178 INFO net.ServerHandle(40) - 监听端口：50500
13:58:02,178 INFO net.ServerHandle(41) - Socket(广发银行)加载成功!
13:58:02,180 INFO net.ServerHandle(40) - 监听端口：50700
13:58:02,180 INFO net.ServerHandle(41) - Socket(新农行)加载成功!
13:58:02,182 INFO net.ServerHandle(40) - 监听端口：39871
13:58:02,182 INFO net.ServerHandle(41) - Socket(新建行)加载成功!
13:58:02,183 INFO net.ServerHandle(40) - 监听端口：60700
13:58:02,184 INFO net.ServerHandle(41) - Socket(上海农商行)加载成功!
13:58:02,185 INFO net.ServerHandle(40) - 监听端口：60800
13:58:02,186 INFO net.ServerHandle(41) - Socket(南京银行)加载成功!
13:58:02,187 INFO net.ServerHandle(40) - 监听端口：60810
13:58:02,187 INFO net.ServerHandle(41) - Socket(吉林银行)加载成功!
13:58:02,190 INFO net.ServerHandle(40) - 监听端口：60820
13:58:02,190 INFO net.ServerHandle(41) - Socket(北京银行)加载成功!
13:58:02,192 INFO net.ServerHandle(40) - 监听端口：60871
13:58:02,192 INFO net.ServerHandle(41) - Socket(民生银行)加载成功!
13:58:02,194 INFO net.ServerHandle(40) - 监听端口：60880
13:58:02,194 INFO net.ServerHandle(41) - Socket(哈尔滨银行)加载成功!
13:58:02,195 INFO net.ServerHandle(40) - 监听端口：60850
13:58:02,196 INFO net.ServerHandle(41) - Socket(上饶银行)加载成功!
13:58:02,198 INFO net.ServerHandle(40) - 监听端口：43400
13:58:02,198 INFO net.ServerHandle(41) - Socket(兴业银行)加载成功!
13:58:02,200 INFO net.ServerHandle(40) - 监听端口：60870
13:58:02,200 INFO net.ServerHandle(41) - Socket(江西农商行)加载成功!
13:58:02,201 INFO net.ServerHandle(40) - 监听端口：60890
13:58:02,202 INFO net.ServerHandle(41) - Socket(东莞农商行)加载成功!
13:58:02,203 INFO net.ServerHandle(40) - 监听端口：60830
13:58:02,204 INFO net.ServerHandle(41) - Socket(河北银行)加载成功!
13:58:02,205 INFO net.ServerHandle(40) - 监听端口：60710
13:58:02,206 INFO net.ServerHandle(41) - Socket(广州农商行)加载成功!
13:58:02,208 INFO net.ServerHandle(40) - 监听端口：60900
13:58:02,208 INFO net.ServerHandle(41) - Socket(江苏银行)加载成功!
13:58:02,210 INFO net.ServerHandle(40) - 监听端口：60910
13:58:02,210 INFO net.ServerHandle(41) - Socket(郑州银行)加载成功!
13:58:02,212 INFO net.ServerHandle(40) - 监听端口：50211
13:58:02,212 INFO net.ServerHandle(41) - Socket(交行新一代)加载成功!
13:58:02,214 INFO net.ServerHandle(40) - 监听端口：50212
13:58:02,214 INFO net.ServerHandle(41) - Socket(交行新一代文件传输)加载成功!
13:58:02,215 INFO net.ServerHandle(40) - 监听端口：60920
13:58:02,215 INFO net.ServerHandle(41) - Socket(恒丰银行)加载成功!
13:58:02,216 INFO net.ServerHandle(40) - 监听端口：60720
13:58:02,217 INFO net.ServerHandle(41) - Socket(中国银行新一代)加载成功!
13:58:02,218 INFO net.ServerHandle(40) - 监听端口：60401
13:58:02,218 INFO net.ServerHandle(41) - Socket(辽宁农商银行寿险)加载成功!
13:58:02,219 INFO net.ServerHandle(40) - 监听端口：50800
13:58:02,220 INFO net.ServerHandle(41) - Socket(天津农商行)加载成功!
13:58:02,221 INFO net.ServerHandle(40) - 监听端口：60410
13:58:02,221 INFO net.ServerHandle(41) - Socket(上海银行寿险)加载成功!
13:58:02,223 INFO net.ServerHandle(40) - 监听端口：60840
13:58:02,223 INFO net.ServerHandle(41) - Socket(浙商银行)加载成功!
13:58:02,224 INFO net.ServerHandle(40) - 监听端口：60930
13:58:02,225 INFO net.ServerHandle(41) - Socket(苏州银行)加载成功!
13:58:02,226 INFO net.ServerHandle(40) - 监听端口：60940
13:58:02,227 INFO net.ServerHandle(41) - Socket(无锡农商行)加载成功!
13:58:02,228 INFO net.ServerHandle(40) - 监听端口：60950
13:58:02,228 INFO net.ServerHandle(41) - Socket(顺德农商行)加载成功!
13:58:02,229 INFO net.ServerHandle(40) - 监听端口：60970
13:58:02,230 INFO net.ServerHandle(41) - Socket(九江银行)加载成功!
13:58:02,231 INFO net.ServerHandle(40) - 监听端口：60520
13:58:02,231 INFO net.ServerHandle(41) - Socket(山西晋城银行)加载成功!
13:58:02,232 INFO net.ServerHandle(40) - 监听端口：60971
13:58:02,233 INFO net.ServerHandle(41) - Socket(安徽农信社)加载成功!
13:58:02,234 INFO net.ServerHandle(40) - 监听端口：60980
13:58:02,234 INFO net.ServerHandle(41) - Socket(徽商银行)加载成功!
```

客户端程序是通过传统的Socket流进行读写的，在银保通Socket改造NIO的过程中，要保证现有的客户端不受影响。  

我们启动客户端进行测试，可以看到，客户端在不改造的前提下通过socket流成功读取到了服务端返回信息。  

![image_1bl03vie41k2j1abbhq93381m2c9.png-51.5kB][1]

对应的服务端日志打印如下：  

![image_1bl0441n3pfpsti1h7o14j4t97m.png-50.5kB][2]

我们再通过socketTool工具对多个端口进行测试：  

![image_1bl04dlhiu4vm9cugn1m396p213.png-33kB][3]

服务端日志打印：  

![image_1bl04epjg1nso1seh14dv74phum1t.png-30.8kB][4]

可以看到，NIO非阻塞监听多个端口信息传输已经实现。  


## 问题
那么，改造之后如何无缝接入银保通现有的业务类呢？  
目前银保通的业务类是线程类，通过传输Socket携带信息进行业务处理并返回信息。其实我们NIO的改造就是为了避免产生这么多的业务线程，因此这就涉及到了实际业务类的改造，这将会对银保通最初封装的架构jar包冲突。一是工作量大；二是风险比较大，因为NIO本身就是一个不断遇到问题以及优化的过程。  

## 总结
这次NIO优化算是对银保通进行一次有益的尝试，同时也给我提了个醒，在以后的架构设计过程中，一定要选择最优方案，即使不是最优的，也要考虑架构的扩展性。


  [1]: http://static.zybuluo.com/coldxiangyu/0u8tuevc0trgcuar881r7jb6/image_1bl03vie41k2j1abbhq93381m2c9.png
  [2]: http://static.zybuluo.com/coldxiangyu/95udl4mw1a24ha6xos41zrv2/image_1bl0441n3pfpsti1h7o14j4t97m.png
  [3]: http://static.zybuluo.com/coldxiangyu/enozus4gwtdkf4u8hvb4mmb4/image_1bl04dlhiu4vm9cugn1m396p213.png
  [4]: http://static.zybuluo.com/coldxiangyu/xcpebqx3xtx6y4jx68hn2sad/image_1bl04epjg1nso1seh14dv74phum1t.png
