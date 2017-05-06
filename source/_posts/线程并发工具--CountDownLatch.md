---
title: 线程并发工具--CountDownLatch
date: 2014-10-18 17:04
categories: Java
tags: 
- 多线程
- CountDownLatch
---

倒计时器，某个线程可以等待这个倒计时指向0的时候开始执行：
```java
final CountDownLatch cdl = new CountDownLatch(10);
		for(int i = 0;i<10;i++){
			final int index = i;
			new Thread(new Runnable(){
				@Override
				public void run() {
					try {
						Thread.sleep(new Random().nextInt(5000));
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("thread " + index + " execute complate!");
					cdl.countDown();
				}
			}).start();
		}
		System.out.println("main thread is awaiting...");
		try {
			cdl.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("all thread execute complate.");
```
主线程启动10个线程开始来减少倒计时数量，等到减到0时主线程继续执行，结果如下：
```bash
main thread is awaiting...
thread 0 execute complate!
thread 8 execute complate!
thread 2 execute complate!
thread 4 execute complate!
thread 9 execute complate!
thread 7 execute complate!
thread 5 execute complate!
thread 1 execute complate!
thread 6 execute complate!
thread 3 execute complate!
all thread execute complate.
```

