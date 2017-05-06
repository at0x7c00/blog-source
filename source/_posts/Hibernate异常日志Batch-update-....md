---
title: Hibernate异常日志Batch update ...
date: 2011-04-01 20:16
categories: 
- Hibernate
tags: 
- Hibernate
- Java
---
```bash
Batch update returned unexpected row count from update [0]; actual row count: 0; expected: 1
```

错误原因： 
    实体主键设置了自增长方式，但是在保存的时候却为它设置了ID主键。 