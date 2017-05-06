---
title: Hibernate Criteria中的三种Distinct
date: 2014-09-17 14:33
categories: Hibernate
tags: distinct
---

# 案例：
```java
/**
 * 客户拜访计划
 * 
 **/
@Entity
@Table(name = "cus_visit")
public class Visit {
	/**
	 * 同行人
	 */
	private Set<Employee> partners;
	@ManyToMany(targetEntity = Employee.class, cascade = { CascadeType.MERGE },fetch = FetchType.LAZY)
	@Cascade(value = { org.hibernate.annotations.CascadeType.SAVE_UPDATE,org.hibernate.annotations.CascadeType.DELETE })
	@JoinTable(name = "cus_visit_employee", joinColumns = { @JoinColumn(name = "visit_id") }, inverseJoinColumns = { @JoinColumn(name = "employee_id") })
	@Fetch(FetchMode.SELECT)
	public Set<Employee> getPartners() {
		return partners;
	}
	public void setPartners(Set<Employee> partners) {
		this.partners = partners;
	}
}
```
Criteria创建：
```java
Criteria criteria = getSession().createCriteria(Visit.class)
				.setFirstResult(pageInfo.getStartIndex())
				.setMaxResults(pageInfo.getNumPerPage());
```

查询条件
```java
private void addQueryCause(Criteria criteria, Visit visit) {
criteria.createAlias("partners", "partner");
if(visit.getPartners()!=null){
			Set<Employee> partnerSet = visit.getPartners();
			for(Employee employee :partnerSet){
				criteria.add(Restrictions.or(Restrictions.like(
						"partner.empNo", employee.getEmpNo(),MatchMode.ANYWHERE),Restrictions.like(
						"partner.name", employee.getName(),MatchMode.ANYWHERE)));
				}
			
		}
//其他查询条件...
}
```

# 第一种：对查询完毕之后的结果进行Distinct
```java
addQueryCause(idsOnlyCriteria, visit);
criteria.setResultTransformer(criteria.DISTINCT_ROOT_ENTITY);
return criteria.list();
```


# 第二种：只查询一个属性，并对这个属性进行Distinct
```java
addQueryCause(idsOnlyCriteria, visit);
criteria.setProjection(Projections.distinct(Projections.id()));
return criteria.list();
```



# 第三种：使用子查询，实现整条记录的Distinct
```java
DetachedCriteria idsOnlyCriteria = DetachedCriteria.forClass(Visit.class);
idsOnlyCriteria.setProjection(Projections.distinct(Projections.id()));
addQueryCause(idsOnlyCriteria, visit);

criteria.add(Subqueries.propertyIn("id", idsOnlyCriteria));

return criteria.list();
```

第三种实现依赖于第二种功能，把条件都放到对id查询的限制上，之后在查询主记录只需要使主键IN子查询结果就行了。最后生成的SQL语句：
```sql
    select
        this_.id as id1_61_0_,
       ....
    from
        cus_visit this_ 
    where
        this_.id in (
            select
                distinct this_.id as y0_ 
            from
                cus_visit this_ 
            left outer join
                cus_visit_employee partners3_ 
                    on this_.id=partners3_.visit_id 
            left outer join
                core_employee partner1_ 
                    on partners3_.employee_id=partner1_.emp_no 
            where
                (
                    this_.planer=? 
                    or partner1_.emp_no=?
                )
        ) 
    order by
        this_.id asc limit ?
```


