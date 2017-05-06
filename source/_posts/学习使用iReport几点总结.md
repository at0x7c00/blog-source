---
title: 学习使用iReport几点总结
date: 2014-10-18 18:56
categories: 报表
tags: ireport
---
title: 学习使用iReport几点总结 date: 2014-10-18 18:56 categories:
tags:
1. iReport和jasperreport之间的关系
个人理解的，iReport仅是一个报表设计器，他所能产生的结果就是jrxml文件，即报表设计木板文件。具体生成为报表，如pdf，word的时候，需要依赖于jasperreport库。
2. 使用流程：
![](http://img.blog.csdn.net/20141018184142550?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这是官方文档中的插图，说明了各个文件之间的转换关系。首先使用iReport生成jrxml文件，然后使用jasperreport library提供的api来处理jrxml文件，直到生成报表。

3. jasperreport在操作pdf的时候依赖于itext-pdf，在支持中文字体的时候依赖于iTextAsian
4. 许多需求在官方提供的下载包里都有例子程序，应该多查看这些例子是如何实现的。
5. 子报表：稍微复杂一点儿的报表都会设计到子报表，从一个主报表出发，可能包含若干个子报表，主报表可以想子报表传递参数，子报表也能给主报表返回参数。
6. 数据源：每个报表都有一个数据源，这个数据源可能是数据库查询、Java集合(Collection)、空(JREmptyDatasource)
7. 报表目标的组成：报表分为多个水平分割的部分，例如Title(标题)，只在报表中显示一次，另外还有Page Header,Column Header等，最重要的是Detail，它表示对于每一条数据要显示的内容。
8. 参数、变量、字段：报表是可以接受传递的参数的，并且可以设置参数的类型。如果外面传递了一个参数age，为了使用这个参数，你需要手动增加一个参数以便使用。
字段是指数据源中每条数据身上的字段。如果你创建的是一个数据库查询的报表，这些字段是自动创建好的，否则，例如一个自定义的类，你需要手动创建好这些类对应的字段。变量，这里包含了与页面相关的一些数据，例如页码、记录index等。
9. Scriptlet：你可以根据接口定义好Scriptlet，然后在iReport中使用。和在Java中使用Bean对象完全一样。