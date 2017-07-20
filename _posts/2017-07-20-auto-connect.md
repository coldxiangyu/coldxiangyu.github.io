---
layout: post
title:  "工作环境实现上网账号自动登录"
date:   2017-07-20 13:36:22
author: coldxiangyu
categories: java
tags: Http
mathjax: true
---

* content
{:toc}


## 背景

许多公司是对上网权限是进行限制的。  

公司对内部员工分配上网账号，员工通过浏览器登陆验证身份才得以联网。  

这样既保证了自身网络的安全性，也实现了对员工上网行为的规范，算是一举两得。  

这也就使得我每天打卡上班开机之后第一件事就是先登陆上网账号进行联网，麻烦的很，我是受不了这种的。所以，何不实现一个开机自动登陆上网账号的脚本呢。而且想来也十分简单，只是模拟一个http post请求而已，再写一个bat脚本文件设置开机启动即可。  

## 实现过程

首先我们要来看一下上网登陆界面：  




![image_1blfc2s7r10h71k8799tbh6gkjm.png-21.3kB][1]  

我们只需要查看源码获取用户名和密码的域ID，作为post请求参数即可。  

![image_1blfcbtv61vrb17197j197tg0nm.png-11.6kB][2]  

由此我们获取到了用户名：username，密码：passwd  
接下来就是通过java自己的http或者HttpClient实现post请求了，这里我采用了java自带的http，不需要引入额外jar包，比较基础，也比较方便。  

源码比较简单，不做过多解释，如下：  

``` java
package com.lxy.http;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLConnection;

public class LoginAB {
	private static final String USER_NAME = "USER_NAME";
	private static final String PASSWORD = "PASSWORD";
	private static final String URL = "http://..../webAuth/index.htm";
	
	public LoginAB(){
		
	}
	
    public static String sendPost() {
    	StringBuffer params = new StringBuffer();
		params.append("username=").append(USER_NAME).append("&").append("passwd=").append(PASSWORD);
        PrintWriter out = null;
        BufferedReader in = null;
        String result = "";
        try {
            URL realUrl = new URL(URL);
            URLConnection conn = realUrl.openConnection();
            conn.setRequestProperty("accept", "*/*");
            conn.setRequestProperty("connection", "Keep-Alive");
            conn.setRequestProperty("user-agent",
                    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            conn.setDoOutput(true);
            conn.setDoInput(true);
            out = new PrintWriter(conn.getOutputStream());
            out.print(params.toString());
            out.flush();
            in = new BufferedReader(
                    new InputStreamReader(conn.getInputStream(),"utf-8"));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
        } catch (Exception e) {
            System.out.println("发送 POST 请求出现异常！"+e);
            e.printStackTrace();
        }
        finally{
            try{
                if(out!=null){
                    out.close();
                }
                if(in!=null){
                    in.close();
                }
            }
            catch(IOException ex){
                ex.printStackTrace();
            }
        }
        return result;
    }    
    
	public static void main(String[] args){
		System.out.println(sendPost());
	}
}

```
我们对此进行试运行，结果如下：  
```
<html><head><meta http-equiv="Content-Type" content="text/html; charset=utf-8"><title></title></head><script>var type="authfailed"var title="认证失败！"var tips="用户名：USER_NAME<br>IP地址：....<br>该IP已登录，请先注销"</script><frameset border=0 cols="0,*" frameborder=0 framespacing=0><frame src="" noresize> <frame src="/authresult/authresult.htm" frameborder=0 noresize scrolling=auto></frameset></html>
```
由于此时我的账号是已经登录的，因此虽然认证失败，但是我们的请求是成功的。  

接下来就是bat文件的编写了，bat无非就是执行我们刚刚编译好的class文件。如下：  
``` dos
@echo off
echo "Load system setting..."
echo ------------------------------------------------------

REM #set java environment
set JAVA_HOME=C:/Program Files/Java/jdk1.7.0_79
echo JAVA_HOME=%JAVA_HOME%

set HTTP_HOME=D:/workspace/mywork/bin
echo HTTP_HOME=%HTTP_HOME%

set CLASSPATH=%HTTP_HOME%
echo CLASSPATH=%CLASSPATH%

REM #out java version
"%JAVA_HOME%/bin/java" -version

REM #set jvm arg
set JVM_ARG=-Xms64M -Xmx64M
echo JVM_ARG=%JVM_ARG%

echo ------------------------------------------------------
echo.

"%JAVA_HOME%/bin/java" %JVM_ARG% com.lxy.http.LoginAB
ping /n 5 127.0.0.1 >nul

```
bat的编写也十分简单，设置class目录以及本地运行的jdk环境，通过`java`命令启动程序即可。  

bat最后一行`ping /n 5 127.0.0.1 >nul`是用来控制bat运行窗口停留时长的，我设置的5s，是为了方便观察登陆结果，如果不想看的话，将其注掉即可。  

接下来就是把此bat文件设置为windows开机启动项了，我用最简单的办法，直接将此bat文件扔进用户启动目录就可以了。我的用户启动目录为：`C:\Users\coldxiangyu\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`  

之后，你就可以重启试一下效果了。  

## 总结

很多时候，用到的技术都是简单的，实现以上的功能也就半小时不到，关键是你有没有一个想法。


  [1]: http://static.zybuluo.com/coldxiangyu/jpbb7hxj9t7p984db6iux0cn/image_1blfc2s7r10h71k8799tbh6gkjm.png
  [2]: http://static.zybuluo.com/coldxiangyu/54s4wxxxwqal0zunyro4jzcu/image_1blfcbtv61vrb17197j197tg0nm.png
