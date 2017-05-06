---
title: 嵌入式(embed)Tomcat的使用
date: 2014-10-24 16:55
categories: Tomcat
tags: 
---
title: 嵌入式(embed)Tomcat的使用 date: 2014-10-24 16:55 categories: Tomcat
tags:
本来想用InstallAnyWhere来制作一个安装文件，里面包含一个tomcat，结果发现InstallAnyWhere一时半会儿可能学不下来。我的目的是想用Java SWT做一个壳子，内嵌一个浏览器，这个浏览器访问tomcat应用，让整个程序看起来像是一个CS架构的。我需要的功能倒是不复杂，想想能不能自己实现一个tomcat。Tomcat有没有提供这样的功能呢，不小心搜索了一下“嵌入式Tomcat”，还真有，并且在Tomcat5的时候就已经支持了：
[https://devcenter.heroku.com/articles/create-a-java-web-application-using-embedded-tomcat#create-your-pom-xml](https://devcenter.heroku.com/articles/create-a-java-web-application-using-embedded-tomcat#create-your-pom-xml)
他用的maven，直接到官网上[下载embed版的压缩包](http://apache.fayea.com/apache-mirror/tomcat/tomcat-7/v7.0.56/bin/embed/apache-tomcat-7.0.56-embed.zip)下来解压放到自己项目的lib目录中，并添加到buildpath中。
按这上面的说法抄一个吧，代码很简单：
```java
public class Main {

    public static void main(String[] args) throws Exception {

        String webappDirLocation = "src/main/webapp/";
        Tomcat tomcat = new Tomcat();

        //The port that we should run on can be set into an environment variable
        //Look for that variable and default to 8080 if it isn't there.
        String webPort = System.getenv("PORT");
        if(webPort == null || webPort.isEmpty()) {
            webPort = "8080";
        }

        tomcat.setPort(Integer.valueOf(webPort));

        tomcat.addWebapp("/", new File(webappDirLocation).getAbsolutePath());
        System.out.println("configuring app with basedir: " + new File("./" + webappDirLocation).getAbsolutePath());

        tomcat.start();
        tomcat.getServer().await();
    }
}
```
按照该文章的步骤完全可以得出正确结果，但是他的方法生成了一个很复杂的bat文件，搞半天不还是搞出了一个和tomcat一模一样的东西来。但是如果直接run这个main方法，会报下面的错误：
```bash
org.apache.jasper.JasperException: java.lang.IllegalStateException: No Java compiler available
	org.apache.jasper.servlet.JspServletWrapper.handleJspException(JspServletWrapper.java:585)
	org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:391)
	org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:395)
	org.apache.jasper.servlet.JspServlet.service(JspServlet.java:339)
	javax.servlet.http.HttpServlet.service(HttpServlet.java:727)
	org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
```

追踪源码，发现是在load类org.apache.jasper.compiler.JDTCompiler的时候出了问题。但是这个类确实已经存在于编译路径中了，咋整？直接在main方法中在启动tomcat之前load一把试试：
```java
 Class.forName("org.apache.jasper.compiler.JDTCompiler");
```
结果：
```bash
Caused by: java.lang.ClassNotFoundException: org.eclipse.jdt.internal.compiler.env.INameEnvironment
	at java.net.URLClassLoader$1.run(Unknown Source)
	at java.net.URLClassLoader$1.run(Unknown Source)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(Unknown Source)
	at java.lang.ClassLoader.loadClass(Unknown Source)
	at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
	at java.lang.ClassLoader.loadClass(Unknown Source)
	... 3 more
```
发现，其实是没有上面的这个类。从[网上找到](http://central.maven.org/maven2/org/eclipse/jdt/core/compiler/ecj/4.4/ecj-4.4.jar)，并加入到编译路径下就ok了：

![](http://img.blog.csdn.net/20141024165118266?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

接下来，把webapp换成真实项目的路径，该项目中使用了spring、Hibernate等工具：
```java
public class Main {

    public static void main(String[] args) throws Exception {

        String webappDirLocation = "src/main/webapp/";
        Tomcat tomcat = new Tomcat();

        //The port that we should run on can be set into an environment variable
        //Look for that variable and default to 8080 if it isn't there.
        String webPort = System.getenv("PORT");
        if(webPort == null || webPort.isEmpty()) {
            webPort = "8080";
        }

        tomcat.setPort(Integer.valueOf(webPort));

//      tomcat.addWebapp("/", new File(webappDirLocation).getAbsolutePath());
        tomcat.addWebapp("/nea", "D:\\apache-tomcat-7.0.34\\webapps\\nea");
        System.out.println("configuring app with basedir: D:\\apache-tomcat-7.0.34\\webapps\\nea");

        tomcat.start();
        tomcat.getServer().await();
    }
}
```

重新启动，发现spring mvc 的路径映射、Hibernate数据库初始化等动作正常执行了。 
