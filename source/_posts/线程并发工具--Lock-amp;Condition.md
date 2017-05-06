---
title: 线程并发工具--Lock和Condition
date: 2014-10-18 10:57
categories: Java
tags: 
- 多线程
---
Lock和Condition是为了替代synchroinzed、wait、notify的，有点是更面向对象，功能上也更强大。下面是一个简单的例子：
```java
class Outputer {
  Lock lock = new ReentrantLock();
  public void output(String str) {
   lock.lock();
   try{
    for (char c : str.toCharArray()) {
     try {
      Thread.sleep(50);
     } catch (InterruptedException e) {
      e.printStackTrace();
     }
     System.out.print(c);
    }
    System.out.println();
   }finally{
    lock.unlock();
   }
  }
 }
```
Lock比传统的synchronzied的优点，除了更面向对象外，它还增加了读写锁的功能。 
读写锁的优点是，多个读的线程是仍然可以并发的，仅仅是读和写、写和写之间做同步，这样既提高了读时的效率也保证了写时的安全：
```java 
ReentrantReadWriteLock wrl = new ReentrantReadWriteLock();
wrl.readLock().lock();
wrl.readLock().unlock();

wrl.writeLock().lock();
wrl.writeLock().unlock();
```


读写锁的一个应用是在缓存的实现中，读取缓存的时候先上读锁，然后判断在缓存中是否有数据，如果没有，则需要写，这个时候需要先将读锁解掉，然后上写锁，再加载数据，之后再解写锁，上读锁，在官方文档中有一个这样的例子：

![](http://img.blog.csdn.net/20141018105326183?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Lock与synchronzied对应，与wait()和notify()对应的则是Condition：
你在将出传统的syncrhonized改成Lock之后，会发现wait和notify没法调了，这话时候需要使用Condition：
```java
Condition condition = lock.newCondition();
condition.await();//this.wait();
condition.signal();//this.notify();
```

Condition较之以前的lock的好处是，之前的lock.wait()和lock.notify()无法根据实际情况去等待和唤醒一个想要的线程。有了Condition之后，可以设置多个condition，线程等待或唤醒是确定的某一个condition，这样在通信上更明确，也更方便了。