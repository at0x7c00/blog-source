---
title: 设计模式-适配器(Adaper)模式
date: 2014-07-28 21:33
categories: 设计模式
tags: 适配器
---

适配器模式中分为类适配器和对象适配器。
类适配器
继承手头现在有的类，通过调用父类（现有类）的方法来实现客户端需要的接口。
客户端想要的接口：
```java
package com.oozinoz.simulation;

/**
 * 火箭模拟
 * 这是一个客户端已经使用的接口
 */
public interface RocketSim {

	double getMass();
	double getThrust();
	void setSimTime(double t);
}
```
手头现有的实现：
```java
package com.oozinoz.physical;

/**
 * 手头上已经有的火箭实现
 */
public class PhysicalRocket {
	double burnArea;
	double burnRate;
	double fuelMass;
	double totalMass;
	public PhysicalRocket(double burnArea, double burnRate, double fuelMass,double totalMass) {
		this.burnArea = burnArea;
		this.burnRate = burnRate;
		this.fuelMass = fuelMass;
		this.totalMass = totalMass;
	}

	public double getBurnTime(){
		//一些具体实现...
		return 0d;
	}
	
	public double getMass(double t){
		//...
		return 0d;
	}
	
	public double getThrust(double t){
		//...
		return 0d;
	}
	
}
```

通过继承的方式来适配：
```java
package com.oozinoz.firework;

import com.oozinoz.physical.PhysicalRocket;
import com.oozinoz.simulation.RocketSim;

/**
 * 适配器
 * 继承现有类，实现客户端接口，将现有实现包装（适配）成客户端想要调用的接口
 */
public class OozinozRocket extends PhysicalRocket implements RocketSim {

	private double time;
	
	public OozinozRocket(double burnArea, double burnRate, double fuelMass,double totalMass) {
		super(burnArea,burnRate,fuelMass,totalMass);
	}

	@Override
	public double getMass() {
		return getMass(time);
	}

	@Override
	public double getThrust() {
		return getThrust(time);
	}

	@Override
	public void setSimTime(double t) {
		this.time = t;
	}

}
```
对象适配器
对象适配器和类适配器不同的是，不通过继承的方式来调用现有的实现，而是通过包装一个现有类对象，调用这个对象的方法来使用现有的实现。
```java
package com.oozinoz.simulation;
import com.oozinoz.physical.PhysicalRocket;

public class OozinozRocket implements RocketSim{

	private double time;
	private PhysicalRocket rocket;
	
	public OozinozRocket(double burnArea, double burnRate, double fuelMass,double totalMass) {
		rocket = new PhysicalRocket(burnArea, burnRate, fuelMass, totalMass);
	}
	
	@Override
	public double getMass() {
		return rocket.getMass(time);
	}
	@Override
	public double getThrust() {
		return rocket.getThrust(time);
	}
	@Override
	public void setSimTime(double t) {
		this.time = t;
	}
}
```
Java的IO流API是最好的适配器实现，适配器模式和装饰模式比较相似。适配器模式侧重于适配新的接口，而装饰模式则侧重于对已有对象的扩展。