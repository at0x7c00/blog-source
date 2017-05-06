---
title: Tomcat6 内存配置
date: 2011-10-18 16:53
categories: 
tags: 
---
title: Tomcat6 内存配置 date: 2011-10-18 16:53 categories:
tags:



Tomcat启动内存设置 

其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)是物理内存的1/4。可以利用JVM提供的-Xmn -Xms -Xmx等选项可 
进行设置 
实例，以下给出1G内存环境下java jvm 的参数设置参考： 
```bash
JAVA_OPTS="-server -Xms800m -Xmx800m -XX:PermSize=64M -XX:MaxNewSize=256m -XX:MaxPermSize=128m -Djava.awt.headless=true "
JAVA_OPTS="-server -Xms768m -Xmx768m -XX:PermSize=128m -XX:MaxPermSize=256m -XX:
NewSize=192m -XX:MaxNewSize=384m"
CATALINA_OPTS="-server -Xms768m -Xmx768m -XX:PermSize=128m -XX:MaxPermSize=256m
-XX:NewSize=192m -XX:MaxNewSize=384m"
```

Linux： 
在/usr/local/apache-tomcat-5.5.23/bin目录下的catalina.sh 
添加： 

```bash
JAVA_OPTS='-Xms512m -Xmx1024m'
```

要加“m”说明是MB，否则就是KB了，在启动tomcat时会报内存不足。 
-Xms：初始值 
-Xmx：最大值 
-Xmn：最小值 


Windows 
在catalina.bat最前面加入
```bash 
set JAVA_OPTS=-Xms128m -Xmx350m
```

如果用startup.bat启动tomcat,OK设置生效.够成功的分配200M内存. 
但是如果不是执行startup.bat启动tomcat而是利用windows的系统服务启动tomcat服务,上面的设置就不生效了, 
就是说set JAVAOPTS=-Xms128m -Xmx350m 没起作用.上面分配200M内存就OOM了.. 

windows服务 执行的是bin\tomcat.exe.他读取注册表中的值,而不是catalina.bat的设置. 
解决办法: 
修改注册表HKEYLOCAL_MACHINE\SOFTWARE\Apache Software 
Foundation\Procrun 2.0\Tomcat6\Parameters\JavaOptions

原值为 
```bash
-Dcatalina.home=E:\Tomcat 6.0
-Dcatalina.base=E:\Tomcat 6.0
-Djava.endorsed.dirs=E:\Tomcat 6.0\common\endorsed
-Djava.io.tmpdir=E:\Tomcat 6.0\temp
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.util.logging.config.file=E:\Tomcat 6.0\conf\logging.properties
```

加入 -Xms300m -Xmx350m 

重起tomcat服务,设置生效 
