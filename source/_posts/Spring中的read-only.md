---
title: Spring中的read-only
date: 2011-03-25 12:27
categories: 
- Spring
tags: 
- Spring
- Hibernate
- junit
- 单元测试
---
今天遇到一个奇怪的问题，使用Hibernate执行查询语句的时候，数据中的数据无缘地被改变了。首先想到的是把持久态的对象在Action中改变了属性值，但是没发现有这样的操作。百思不得其解。使用Debug模式跟踪了一下，发现确实是在list方法执行时出现对数据的莫名更改，而我用JUnit进行单元测试时，却没有这样的情况。 

```java
	public List<T> findByHql(String hql, int startPos, int length,Object[] paramValues) {
		session = getCurrentSession();
		Query query = session.createQuery(hql);
		if (paramValues!=null&&paramValues.length>0){
			for (int i = 0; i < paramValues.length; i++) {
				query.setParameter(i, paramValues[i]);
			}
		}
		query.setFirstResult(startPos);
		query.setMaxResults(length);
		List<T> list = query.list();
		return list;
	}

    

```


解决办法，在Spring中的事务控制部分将该方法配置成只读方式： 

```xml
<tx:attributes>
			<tx:method name="add*" propagation="REQUIRES_NEW" />
			<tx:method name="update*" propagation="REQUIRES_NEW" />
			<tx:method name="delete*" propagation="REQUIRES_NEW" />
			<tx:method name="find*" read-only="true" />
			<tx:method name="*" propagation="REQUIRED" />
</tx:attributes>

```