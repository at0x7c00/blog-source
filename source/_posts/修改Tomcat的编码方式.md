---
title: 修改Tomcat的编码方式
date: 2011-03-20 18:49
categories: 
- 运维
tags: 
- Tomcat
---

今天做一个简单的在线音乐播放器，使用的是<embed>标签，但是出现一个问题，即中文音乐文件无法访问。 
开始以为是embed标签的问题。但是以前写过一个静态页面就是使用了中文名称。另外还有一个现象是， 
即使将音乐文件直接放在tomcat的WebRoot目录下，通过浏览器都无法访问，使用迅雷也无法下载。所以就想到了是Tomcat的问题了。 

默认情况下，tomcat使用的是iso8859-1的编码编码方式，浏览器的embed标签中src指向的地址要通过tomcat去解析。如果包含中文，采用这种编码方式就会出现乱码问题，而在这种情况下，乱码问题就表现出无法访问该音频文件了。 

解决方法很简单： 
修改tomcat下的conf/server.xml文件，找到如下代码：    
```xml
   <Connector port="8080" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="8443" /> 
```
   
这段代码规定了Tomcat监听HTTP请求的端口号等信息。可以在这里添加一个属性：URIEncoding，将该属性值设置为UTF-8，即可让Tomcat（默认ISO-8859-1编码）以UTF-8的编码处理get请求。更改后的代码如下所示： 

```xml
<Connector port="8080" 
             URIEncoding="UTF-8" 
             protocol="HTTP/1.1" 
             connectionTimeout="20000" 
             redirectPort="8443" />
```