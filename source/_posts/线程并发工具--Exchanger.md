---
title: 线程并发工具--Exchanger
date: 2014-10-18 17:17
categories: 多线程
tags: Exchanger
---
Exchanger可以实现两个线程之间的数据交换：
```java
		final Exchanger<String> exchanger = new Exchanger<String>();
		new Thread(new Runnable(){
			@Override
			public void run() {
				try {
					Thread.sleep(new Random().nextInt(5000));
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName() + ":i am ready to buy!");
				String give = "money";
				try {
					String goods = exchanger.exchange(give);
					System.out.println(Thread.currentThread().getName() + ": give " + give + " and get " + goods);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}).start();
		
		
		new Thread(new Runnable(){
			@Override
			public void run() {
				try {
					Thread.sleep(new Random().nextInt(5000));
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName() + ":i am ready to sale!");
				String sale = "goods";
				try {
					String money = exchanger.exchange(sale);
					System.out.println(Thread.currentThread().getName() + ": give " + sale + " and get " + money);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}).start();
```

上面的示例模拟了买卖双方的一个交换：
```bash
Thread-1:i am ready to sale!
Thread-0:i am ready to buy!
Thread-0: give money and get goods
Thread-1: give goods and get money
```


