---
title: ognl.OgnlException: target is null for setProperty(null, "description", [Ljava.l
date: 2011-04-16 11:28
categories: 
- Struts
tags: 
- Spring
- Struts
- prototype
- JSP
- Web
---

以前也遇到过这样的错误，没有怎么注意。今天又遇到了。总结一下。 
Struts页面参数传递错误原因有四种可能性。 

一、数据类型不匹配： 
    如Action中为Integer类型，而传递过来的是String类型，此种原因多半因为写Struts标签或EL表达式时写错了。 

二、也可以称作数据类型不匹配： 
    如Action中的属性为Integer id;   但是页面上传递参数时同时传递了多个name为id 
的参数，如此以来即因为不能将整型数组转换成整型而出错。 

三、无法访问属性出错 
   如Action中有一个Student的属性stu,Student有age的属性，但是Student却没有无参数的构造方法，或者age这个属性没有getter或setter方法，就或出错，这是网上的说法。 

四、Spring管理下的Action出错： 
   这是今天我遇到的错误，也是最隐蔽的错误。 
   情况如下： 
    我使用注解的方式，将Action纳入Spring管理： 
```xml
	 	<action name="*_*" class="{1}" method="{2}">
	 	   <!-- 增加或修改的输入页面 -->
	 	   <result name="input">/WEB-INF/pages/{1}/{2}UI.jsp</result>
	 	   
	 	   <!-- 直接访问与action中方法名称对应的页面 -->
		   <result name="success">/WEB-INF/pages/{1}/{2}.jsp</result>
		   
		   <!--重定向到list页面 -->
		   <result name="reload" type="redirectAction">{1}_list.action?pageNum=${pageNum}</result>
		   
		</action>
```

BussinessAction(业务),里面用到了javaBean：AcceptType(受理类型): 
```java
/**
 * @author huqiao 2011-04-11
 *
 */
@Controller("bussiness")
@Scope(value="prototype")
public class BussinessAction extends BaseAction<Bussiness> {
	
	private AcceptType acceptType;

......
}
```

而另外一个Action:AcceptTypeAction: 

```java
/**
 * @author huqiao 2011-04-11
 */
@Controller("acceptType")
@Scope(value="prototype")
public class AcceptTypeAction extends BaseAction<AcceptType> {
	
	private AcceptType acceptType;
......
}
```

即：AcceptTypeAction纳入Spring管理的名字与BussinessAction中的AcceptType属性名称重复了，在Bussiness添加页面提交时报类型转换异常： 
```bash
Cannot convert value of type [cn.chinacti.crm.action.AcceptAction] to required type [cn.chinacti.crm.entity.AcceptType] for property 'acceptType': no matching editors or conversion strategy found
```
解决办法： 
   修改Action的名称