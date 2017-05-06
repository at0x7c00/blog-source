---
title: Hibernate级联关系的说明和优化
date: 2011-10-15 21:16
categories: 
- Hibernate
tags: 
---

今天接到一个公司的电话面试，里面问道Hibernate的级联，很失败，竟然回答的吞吞吐吐的，失败。前几天刚刚做了，看开做东西要仔仔细细，马虎问题大。 

说说级联吧： 

========================INVERSE=============================== 

Hibernate里面的inverse 有两个值 true ,false  ; 

inverse的意思是翻转，这里面理解为对对应表的的维护 [http://space.itpub.net/22259926/viewspace-631423](http://space.itpub.net/22259926/viewspace-631423 )

里面说的，如果inverse为false的化，delete不会修改order表单，即对应关系没有维护。但是新增的时候，会增加order表单， 

下面是转载的， 

========================CASCADE=========================== 


cascade 有五个选项 分别是: all ,delete ,none,save-update,delete-orphan ; 
all : 所有情况下均进行关联操作。 
none：所有情况下均不进行关联操作。这是默认值。 
save-update:在执行save/update/saveOrUpdate时进行关联操作。 
delete：在执行delete时进行关联操作。 
delete-orphan: 当save/update/saveOrUpdate时，相当于save-update ；当删除操作时，相当于delete ； 


all的意思是save-update + delete 
all-delete-orphan 的意思是当对象图中产生孤儿节点时,在数据库中删除该节点 
all比较好理解,举个例子说一下all-delete-orphan: 
Category与Item是一对多的关系,也就是说Category类中有个Set类型的变量items. 
举个例子,现items中存两个Item, item1,item2,如果定义关系为all-delete-orphan 
当items中删除掉一个item(比如用remove()方法删除item1),那么被删除的Item类实例 
将变成孤儿节点,当执行category.update(),或session.flush()时 
hibernate同步缓存和数据库,会把数据库中item1对应的记录删掉 

========================LAZY=========================== 

结论1： HQL代码 > fetch（配置） > lazy （配置） 
结论2： 默认 lazy="true" 
结论3： fetch 和 lazy 主要是用来级联查询的，  

fetch参数指定了关联对象抓取的方式是select查询还是join查询，select方式时先查询返回要查询的主体对象（列表），再根据关联外键id，每一个对象发一个select查询，获取关联的对象，形成n+1次查 询； 而join方式，主体对象和关联对象用一句外键关联的sql同时查询出来，不会形成多次查询。 
如果你的关联对象是延迟加载的，它当然不会去查询关联对象。 另外，在hql查询中配置文件中设置的join方式是不起作用的（而在所有其他查询方式如get、criteria或再关联获取等等都是有效的），会使用select方式，除非你在hql中指定join fetch某个关联对象。fetch策略用于定义 get/load一个对象时，如何获取非lazy的对象/集合。 这些参数在Query中无效。 

========================batch-size============================== 

今天有个问题，就是一对多查询时候，比如Customer里面包含了太多Order表单怎么优化？当时懵了 

其实可以使用Hibernate的延迟加载功能，即时Lazy=true，只有在真正需要的时候，才去从数据库加载“本体”，Customer的某个Order，只用当该Order真正需要时候的，采取从数据库加载 