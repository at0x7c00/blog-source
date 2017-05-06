---
title: 线程并发工具--Semaphore(信号灯)
date: 2014-10-18 11:28
categories: Java
tags: 多线程
---
Semaphore可以运行指定个数的线程同时运行某段代码，而不是一般同步情况下的一个线程。下面的程序中，10个线程运行的时候，都必须先获得到Semaphore，运行完毕之后归还Semaphore，达到限制指定个数的多个线程同时执行的效果。
```java
		final Semaphore sp = new Semaphore(3,false);
		ExecutorService service = Executors.newCachedThreadPool();
		
		for(int i = 1;i<10;i++){
			final int index = i;
			service.execute(new Runnable() {
				@Override
				public void run() {
					try {
						sp.acquire();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(index + ":thread" + Thread.currentThread().getName() + " get the  Semaphore,Semaphore left:" + sp.availablePermits());
					try {
						Thread.sleep(new Random().nextInt(3000));
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					sp.release();
				}
			});
			
		}
		service.shutdown();

```

上面的程序中设置了3个信号灯，但是却有10个线程等待这来获取到信号灯并执行自己的代码。这10个线程必须等待，每次最多只能有3个线程能获取到信号灯并执行。注意到Semaphore的构造方法中的那个boolean参数，它表示线程来获取信号灯的权利是否是按先后顺序的。
true时：
![](http://img.blog.csdn.net/20141018112528466?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


false时可能的结果：

![](http://img.blog.csdn.net/20141018112223171?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


