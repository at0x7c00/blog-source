---
title: Hibernate中的left outer join
date: 2014-12-17 19:17
categories: Hibernate
tags: 
---

首先，最简单的是一对多的连接，比如：
```sql
select student from Teacher t join t.students student where student....
```

如果是多对一呢？这里有隐式和显示的区别（上面的一对多的情况属于隐式连接）。可以像下面这样
```sql
select student from Student student 
where student.teacher.age>30
```

这属于隐式的，Hibernate会自动连接Teacher表。也可以像下面这样显示的连接：
```sql
select student from Student student
left outer join student.teacher t 
where t.age>30
```

这里明确的说明了连接Teacher表的时候要使用左外连接。这里容易忽略outer，如果写成下面的样子：
```sql
select student from Student student
left join student.teacher t 
where t.age>30
```
这是错误的写法，Hibernate将不认识t.age的条件。

