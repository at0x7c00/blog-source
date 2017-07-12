---
title: MySQL创建外键时报Can't create table errno 150 错误解决办法
date: 2016-05-17 02:30
categories: 数据库
tags: MySQL
---

总的来说，这个问题的原因就是创建的外键和关联的表的主键类型不匹配。下面用个简单的例子来说明。
两张很简单的表，学生表和教师表：
```sql
CREATE TABLE `t_teacher` (  
  `id` varchar(11) NOT NULL,  
  `name` varchar(20) DEFAULT NULL,  
  PRIMARY KEY (`id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;  
  
CREATE TABLE `t_student` (  
  `id` varchar(11) NOT NULL DEFAULT '',  
  `name` varchar(20) DEFAULT NULL,  
  PRIMARY KEY (`id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;  
```

现在需要给学生表创建一个外键来关联教师表：
第一种情况，很简单，就是列的类型不匹配，比如用int类型的列去关联varchar的列：
```sql
ALTER TABLE `t_student`  
ADD COLUMN `teacher_id`  int(11) NULL AFTER `name`;  
  
ALTER TABLE `t_student` ADD CONSTRAINT `teacher_id_fk` FOREIGN KEY (`teacher_id`) REFERENCES `t_teacher` (`id`) ON DELETE RESTRICT ON UPDATE RESTRICT;  
```

另外一种情况，虽然数据类型一致了，但是字符编码不一致，例如：




最后一种情况则是，找不到外键引用的列。列在增加外键的时候，可能已经存在数据了（历史数据，或者更新数据库表时设置的默认值），但这些数据并不一定能在外键关联表中找到对应的行记录。这种情况也会导致外键创建失败。