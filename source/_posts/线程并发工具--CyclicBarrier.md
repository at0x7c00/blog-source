---
title: 线程并发工具--CyclicBarrier
date: 2014-10-18 16:45
categories: Java
tags: 
- 多线程
- CyclicBarrier
---

CyclicBarrier能实现的效果是多个线程同时执行，这些线程执行的时间不一。但是要求在某一个点上，需要这些线程都执行完毕了之后，全部线程才能往下执行。 
下面是示例程序：
```java
ExecutorService service = Executors.newCachedThreadPool();
		final CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
		for(int i = 0;i<3;i++){
			Runnable runnable = new Runnable(){
				
				Random random = new Random();
				
				@Override
				public void run() {
					try {
						Thread.sleep(random.nextInt(5000));
						System.out.println("point 1:"+Thread.currentThread().getName()+" execute complate.");
						cyclicBarrier.await();
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
					
					try {
						Thread.sleep(random.nextInt(5000));
						System.out.println("point 2:"+Thread.currentThread().getName()+" execute complate.");
						cyclicBarrier.await();
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
					
					try {
						Thread.sleep(random.nextInt(5000));
						System.out.println("point 3:"+Thread.currentThread().getName()+" execute complate.");
						cyclicBarrier.await();
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
					
					try {
						Thread.sleep(random.nextInt(5000));
						System.out.println("point 4:"+Thread.currentThread().getName()+" execute complate.");
						cyclicBarrier.await();
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
				}
			};
			service.execute(runnable);
			//service.shutdown();
		}
```
执行结果： 
```bash
point 1:pool-1-thread-3 execute complate.
point 1:pool-1-thread-2 execute complate.
point 1:pool-1-thread-1 execute complate.
point 2:pool-1-thread-2 execute complate.
point 2:pool-1-thread-1 execute complate.
point 2:pool-1-thread-3 execute complate.
point 3:pool-1-thread-3 execute complate.
point 3:pool-1-thread-1 execute complate.
point 3:pool-1-thread-2 execute complate.
point 4:pool-1-thread-1 execute complate.
point 4:pool-1-thread-2 execute complate.
point 4:pool-1-thread-3 execute complate.
```
可以看到，到达么一个点的顺序可能不固定，但是没有说一个线程才到达point1的时候，另外一个线程已经达到point2了，即大家共同达到一个点之后才会继续往下执行。 

