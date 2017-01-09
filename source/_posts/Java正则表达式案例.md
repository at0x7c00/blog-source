---
title: Java正则表达式案例
date: 2016-12-02
tags: 正则表达式
categories: 技术浮云
---

# 反向引用
```java
	String content = "[RECORDID=2012233234525.2586]";
	String replaceBy = "<a href=\"../detail.do?recordId=$1\"  title=\"查看详情\">$1</a>";
	
	content = content.replaceAll("\\[RECORDID=(\\d{1,}(.)?\\d{1,})\\]", replaceBy);
	
	//<a href="../detail.do?recordId=2012233234525.2586"  title="查看详情">2012233234525.2586</a>
	System.out.println(content);
```
