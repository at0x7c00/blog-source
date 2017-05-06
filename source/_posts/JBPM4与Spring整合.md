---
title: JBPM4与Spring整合
date: 2011-11-06 14:14
categories: 
- JBPM
tags: 
- JBPM
---
title: JBPM4与Spring整合 date: 2011-11-06 14:14 categories: - JBPM - Spring tags:
- JBPM
之前一直报错,郁闷了很久： 
```bash
[...] 
nested exception is org.hibernate.MappingException: Unknown entity: org.jbpm.pvm.internal.id.PropertyImpl 
at 
[...] 
```
但是，该类的hibernate映射文件是写在jar包里面的。Hibernate为何没有解析到这个配置文件，不得而知。查看jbpm.hibernate.cfg.xml文件： 
```xml
     <mapping resource="jbpm.repository.hbm.xml" />
     <mapping resource="jbpm.execution.hbm.xml" />
     <mapping resource="jbpm.history.hbm.xml" />
     <mapping resource="jbpm.task.hbm.xml" />
     <mapping resource="jbpm.identity.hbm.xml" />
```

PropertyImpl的映射就在第一个被引入进来的文件当中，可以猜想到，其实整个的jbpm.hibernate.cfg.xml文件都没有Hibernate找到。 
查看两种事务控制方式对应的配置文件jbpm.tx.spring.cfg.xml和jbpm.tx.hibernate.cfg.xml，发现后者会默认地去读classpath中的jbpm.hibernate.cfg.xml文件，而使用spring的时候却没有这样的操作(在applicationContext.xml文件中配置嘛)。 
另外，在查看源码的时候，到处可见的是默认配置"jbpm.cfg.xml"： 
```java
/**
 * @author Joram Barrez
 */
public class SpringHelper implements ApplicationContextAware {
  
  protected ApplicationContext applicationContext;
  protected String jbpmCfg = "jbpm.cfg.xml";
...
```

却看不到jbpm.hibernate.cfg.xml的默认值。jbpm的配置文件设计思想逐渐明晰： 
总的配置从jbpm.cfg.xml开始，你可以更改这个文件以便使用自己的事务控制方式。 
如果使用的是hibernate的事务控制，那么你就得提供好jbpm.tx.hibernate.cfg.xml文件，并做好配置。如果使用的是spring的事务控制，只需要在jbpm.cfg.xml文件中切换到spring中即可，其他的关于数据库连接信息，sessionFactory等等由你自己到applicationContext文件中配置即可。 
以下是在applicationContxt.xml文件中配置的关于jbpm信息： 
```xml
	<!-- 业务系统 -->
	<bean id="crmSessionFactory"
		class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
		<property name="dataSource" ref="crmDataSource" />
		<!-- 引入外部配置文件，将jbpm和CRM的配置信息放到一起 -->
		<property name="configLocation" value="classpath:jbpm.hibernate.cfg.xml"></property>
		
		<!-- 扫描指定目录下的所有实体属性映射配置文件 -->
		<property name="mappingDirectoryLocations">
		  <list>
		    <value>classpath:/cn/chinacti/crm/entity</value>
		  </list>
		</property>
	</bean>
```

如此，我的问题得以解决。 
道理很简单，以后记得，与Spring 集成，那么所有的数据库信息都应该从applicationContext.xml配置出发，而不应让集成进来的组件自己去找自己的配置文件。