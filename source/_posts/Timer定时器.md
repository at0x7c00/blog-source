---
title: Timer定时器
date: 2014-10-18 09:21
categories: Java
tags: 定时器
---

最简单的需求，设定多长时间以后执行某个动作:
```java
public class MyTimerTask extends TimerTask{
 @Override
 public void run() {
  System.out.println("task running...");
 }
}
```
每三秒执行一次： 
```java
 Timer timer = new Timer();
 timer.schedule(new MyTimerTask(), 3000);
```
Timer提供不同的API实现不同的定时功能，例如:
1）定时到指定的时间执行（一次）
2）定时到指定的时间后每隔一段时间执行一次（循环多次）
3）延迟指定时间后执行
API如图所示：
![API](http://img.blog.csdn.net/20141018092150231?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

另外，如果希望动态修改执行间隔，可以先调用cancel();方法取消定时器，然后重新schedule();

比较复杂一点儿的定时任务可以参考[Quzrtz](http://www.quartz-scheduler.org/documentation/best-practices)。