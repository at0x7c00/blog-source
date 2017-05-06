---
title: [转]JXL导出Excel
date: 2011-09-18 09:43
categories: 
- 报表
tags: 
---

一、 数据格式化 
在Excel中不涉及复杂的数据类型，能够比较好的处理字串、数字和日期已经能够满足一般的应用。 

1、 字串格式化 
    字符串的格式化涉及到的是字体、粗细、字号等元素，这些功能主要由WritableFont和WritableCellFormat类来负责。假设我们在生成一个含有字串的单元格时，使用如下语句，为方便叙述，我们为每一行命令加了编号： 
```java
WritableFont font1= new WritableFont(WritableFont.TIMES,16,WritableFont.BOLD); ① 
```

或//设置字体格式为excel支持的格式 

```java
WritableFont font2=new WritableFont(WritableFont.createFont("楷体_GB2312"),12,WritableFont.NO_BOLD ); 

WritableCellFormat format1=new WritableCellFormat(font1);② 

Label label=new Label(0,0,”data 4 test”,format1) ③ 

```

其中①指定了字串格式：字体为TIMES，字号16，加粗显示。WritableFont有非常丰富的构造子，供不同情况下使用，jExcelAPI的java-doc中有详细列表，这里不再列出。 ②处代码使用了WritableCellFormat类，这个类非常重要，通过它可以指定单元格的各种属性，后面的单元格格式化中会有更多描述。 ③处使用了Label类的构造子，指定了字串被赋予那种格式。在WritableCellFormat类中，还有一个很重要的方法是指定数据的对齐方式，比如针对我们上面的实例，可以指定： 

```java
//把水平对齐方式指定为居中 

format1.setAlignment(jxl.format.Alignment.CENTRE); 

//把垂直对齐方式指定为居中 

format1.setVerticalAlignment(jxl.format.VerticalAlignment.CENTRE); 

//设置自动换行 

format1.setWrap(true); 
```


二、单元格操作 
Excel中很重要的一部分是对单元格的操作，比如行高、列宽、单元格合并等，所幸jExcelAPI提供了这些支持。这些操作相对比较简单，下面只介绍一下相关的API。 

1、 合并单元格 
```java
WritableSheet.mergeCells(int m,int n,int p,int q); 
```


作用是从(m,n)到(p,q)的单元格全部合并，比如： 

```java
WritableSheet sheet=book.createSheet(“第一页”,0); 
//合并第一列第一行到第六列第一行的所有单元格 
sheet.mergeCells(0,0,5,0); 
```

合并既可以是横向的，也可以是纵向的。合并后的单元格不能再次进行合并，否则会触发异常。 

2、 行高和列宽 
```java
WritableSheet.setRowView(int i,int height); 
```


作用是指定第i+1行的高度，比如： 

```java
//将第一行的高度设为200 
sheet.setRowView(0,200); 
WritableSheet.setColumnView(int i,int width); 
```


作用是指定第i+1列的宽度，比如： 

```java
//将第一列的宽度设为30 
sheet.setColumnView(0,30); 
```


三、操作图片 

```java
public static void write()throws Exception{ 

WritableWorkbook wwb=Workbook.createWorkbook(new File("c:/1.xls")); 

WritableSheet ws=wwb.createSheet("Test Sheet 1",0); 

File file=new File("C:\\jbproject\\PVS\\WebRoot\\weekhit\\1109496996281.png"); 

WritableImage image=new WritableImage(1, 4, 6, 18,file); 

ws.addImage(image); 

wwb.write(); 

wwb.close(); 

} 
```


很简单和插入单元格的方式一样，不过就是参数多了些，WritableImage这个类继承了Draw，上面只是他构造方法的一种，最后一个参数不用说了，前面四个参数的类型都是double，依次是 x, y, width, height,注意，这里的宽和高可不是图片的宽和高，而是图片所要占的单位格的个数，因为继承的Draw所以他的类型必须是double，具体里面怎么实现的我还没细看：）因为着急赶活，先完成功能，其他的以后有时间慢慢研究。以后会继续写出在使用中的心得给大家。 
