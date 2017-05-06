---
title: Hibernate中的LEFT JOIN
date: 2011-03-21 21:31
categories: 
- Hibernate
tags: 
- Hibernate
- Java
---

```java
public class Order {
   private Set<OrderProduct> orderProducts=new HashSet<OrderProduct>();     
}
public class OrderProduct {
   private Order order;// 订单
}
```

```sql
Query query1=session.createQuery("SELECT count(*),SUM(o.sum),op.product FROM Order o LEFT JOIN o.orderProducts op  WHERE op.product.id=18 GROUP BY op.product");
```

如上，首先，订单与订单产品之间是一对多关联关系，orderProduct是Order的一个属性，这样，表外连接时，就不再使用ON子句了。 

而采用以下查询语句：
```sql 
Query query1=session.createQuery("FROM Order o LEFT JOIN o.orderProducts op WHERE o.isReturn = 0 and o.isdel = 0 
");
```

查询的结果将为一个类型为数组的List，其中，第一个元素类型为Order，第二个元素为OrderProduct