---
title: Hibernate One to One
date: 2014-07-18 15:25
categories: Hibernate
tags: 
---

之前一直用多对一，即使是一对一也转换成多对一方式解决。今天逃不了要用一对一还是费了点功夫才搞定。
多对一关联关系的配置上，在两边分配配置多对一和一对多就行了。但是一对一的时候，两边都是@OneToOne，那么这个外键究竟会放到哪个表里面呢？当然，在实际意义来说，放在哪个表里面都合理。但是在Hibernate的配置上就应该是确定在某一张表的。在JPA的官方文档中有这样的说明：

>mappedBy
>public abstract java.lang.String mappedBy
>(Optional) The field that owns the relationship. This element is only specified >on the inverse (non-owning) side of the association.

意思是说，哪一方是inverse就配置mappedBy就行了，这样，外键就产生在了非inverse一方对应的表里了。
以员工（Employee）和薪酬信息（SalaryInfo）为例说明，这里希望外键产生在薪酬信息表里面：
员工：
```java
@Entity
@Table(name = "core_employee")
public class Employee {
   private SalaryInfo salaryInfo;
	@OneToOne(mappedBy="empNo")
	public SalaryInfo getSalaryInfo() {
		return salaryInfo;
	}
...
}
```
   

薪酬信息：
```java
@Entity
@Table(name = "sal_salary_info")
public class SalaryInfo {
	private Employee emp;
	@OneToOne(cascade=CascadeType.ALL,targetEntity=Employee.class)
	@JoinColumn(name = "emp_no", nullable = false)
	public Employee getEmp() {
		return this.emp;
	}
...
}
```
另外，在Hibernate的Criteria的查询中，不支持复杂对象的比较，比如下面的代码：
```java
Criteria criteria = getSession().createCriteria(Employee.class);
		criteria.add(Restrictions.isNull("salaryInfo"));
```
意图查询没有薪酬信息的员工，结果在执行查询的时候，将产生下面的语句：
```sql
select * 
		from employee emp
		inner join salaryinfo s on s.emp_no=emp.emp_no
		where emp.emp_no is null
```
很是奇怪为啥条件是emp.emp_no is null。应该写成下面的形式：
```java
Criteria criteria = getSession().createCriteria(Employee.class);
		criteria.add(Restrictions.isNotNull("salaryInfo.id"));
```
除了isNotNull，其他的所有方法都是同样的道理，在进行复杂类型属性比较的时候需要点到里面的属性才会触发生成正确的sql查询条件。

上面的代码到现在这个时候还无法正确查询到结果。因为Criteria的默认表连接方式为inner join，这样的连接方式导致没有找到薪酬信息的员工记录也无法出现在查询结果中。需要显示声明表连接方式：
```java
Criteria criteria = getSession().createCriteria(Employee.class);
		criteria.add(Restrictions.isNotNull("salaryInfo.id",JoinType.LEFT_OUTER_JOIN));
```	

