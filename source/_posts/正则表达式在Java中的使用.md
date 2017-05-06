---
title: 正则表达式在Java中的使用
date: 2014-11-16 17:51
categories: Java
tags: 正则表达式
---

这里不打算说明正则表达式的具体细节，只讲Java中使用正则表达式的一些基础知识。
一个简单的例子，使用正则表达式来匹配电话号码，电话号码包括了三到四位的区号；可有可无的连接符；6位到八位的电话号码。正则表达式如下：
```bash
\d{3,4}-?\d{6,9}
```
这里仅仅是举个例子，实际使用中，考虑到电话号码的合法性，区号还需要改进。简单而言，像下面这样就能使用这个正则表达式对字符串进行验证了：
```java
boolean match = Pattern.matches("\\d{3,4}-?\\d{6,9}", "010-23232222");
```
match就是我们需要的验证结果。
也可以转换成Matcher对象再调用Matcher的matches方法：
```java
Pattern pattern = Pattern.compile("\\d{3,4}-?\\d{6,9}");
//boolean match = Pattern.matches("\\d{3,4}-?\\d{6,9}", "010-23232222");
Matcher matcher = pattern.matcher("010-23232222");
System.out.println(matcher.matches());
```

这里，仅仅是验证一个字符串是否符合正则表达式，这是最简单的应用。如果想要在在一个字符串中进行匹配查找，就需要使用到Matcher对象了：
```java
Pattern pattern = Pattern.compile("\\d{3,4}-?\\d{6,9}");
Matcher matcher = pattern.matcher("联系电话：010-23232222，地址：北京市朝阳区...");
while(matcher.find()){
	System.out.println(matcher.group());
}
```
这样，就能从字符串中摘取出符合正则表达式的内容了。

# 匹配模式
有的时候，需要进行忽略大小写的匹配：
```java
String moneyRegex = "[+-]?(\\d)+(.(\\d)*)?(\\s)*[CF]";
String input = "-3.33c";
boolean match = Pattern.compile(moneyRegex,Pattern.CASE_INSENSITIVE).matcher(input).matches();
```
该例子实现匹配摄氏温度和华氏温度，对于以C、c、F和f结尾的温度值都能匹配。
这是依赖于API来忽略大小写，正则表达式内部也支持忽略大小写：
```java
String moneyRegex = "[+-]?(\\d)+(.(\\d)*)?(\\s)*(?i)[CF]";
String input = "-3.33f";
boolean match = Pattern.compile(moneyRegex).matcher(input).matches();
```
Java还提供了很多类似于CASE_INSENSITIVE的模式标记，多个模式标记可以使用“|”符号连接。

# Pattern.CANON_EQ
启用正则等价。

# Pattern.COMMENTS
启用注释，开启之后，正则表达式中的空格以及#号行将被忽略：
```java
String comments = "    (\\d)+#this is comments.";
String input = "1234";
boolean match = Pattern.compile(comments,Pattern.COMMENTS).matcher(input).matches();//true
boolean match2 = Pattern.compile(comments).matcher(input).matches();//false
```

可以看到，#号到行尾的注释部分和前面的空白字符都被忽略了。正则表达式内置的启用注释为（?x）：
```java
String comments = "(?x)    (\\d)+#this is comments.";
String input = "1234";
boolean match = Pattern.compile(comments,Pattern.COMMENTS).matcher(input).matches();//true
boolean match2 = Pattern.compile(comments).matcher(input).matches();//true
```
# Pattern.DOTALL
启用dotall模式，一般情况下，点号（.）匹配任意字符，但不匹配换行符，启用这个模式之后，点号还能匹配换行符：
```java
String dotall = "<xml>(.)*</xml>";
String dotallInput = "<xml>\r\n</xml>";
boolean match = Pattern.compile(dotall,Pattern.DOTALL).matcher(dotallInput).matches();//true
boolean match2 = Pattern.compile(dotall).matcher(dotallInput).matches();//false
```

使用内置方式：
```java
String dotall = "(?s)<xml>(.)*</xml>";
String dotallInput = "<xml>\r\n</xml>";
boolean match = Pattern.compile(dotall,Pattern.DOTALL).matcher(dotallInput).matches();//true
boolean match2 = Pattern.compile(dotall).matcher(dotallInput).matches();//true
```

# Pattern.LITERAL
平白字符模式，启用这个模式之后，所有元字符、转义字符都被看成普通的字符，不再具有其他意义：
```java
System.out.println(Pattern.compile("\\d",Pattern.LITERAL).matcher("\\d").matches());//true
System.out.println(Pattern.compile("\\d",Pattern.LITERAL).matcher("2").matches());//false
		
System.out.println(Pattern.compile("(\\d)+",Pattern.LITERAL).matcher("1234").matches());//false
System.out.println(Pattern.compile("(\\d)+").matcher("1234").matches());//true
		
System.out.println(Pattern.compile("(\\d){2,3}",Pattern.LITERAL).matcher("(\\d){2,3}").matches());//true
```

# Pattern.MULTILINE
多行模式：默认情况下，输入的字符串被看作是一行，即便是这一行中包好了换行符也被看作一行。当匹配“^”到“$”之间的内容的时候，整个输入被看成一个一行。启用多行模式之后，包含换行符的输入将被自动转换成多行，然后进行匹配：
```java
String multiline = "^(\\d)+$";
String multilineInput = "1234\r99\rabc\r128";
		
Pattern p = Pattern.compile(multiline, Pattern.MULTILINE);
Matcher m = p.matcher(multilineInput);
while(m.find()){
	System.out.println(m.group());
}
```
例子中，将字符串中的数字摘取出来，如果不启用多行模式，将什么也匹配不到。可以用(?m)在正则表达式内部启用多行模式。
# Pattern.UNIX_LINE
unix行模式，大多多数系统的行都是以“\n”结尾的，但是少数系统，比如Windows，却是以“\r\n”组合来结尾的，启用这个模式之后，将会只以“\n”作为行结束符，这会影响到^、$和点号(点号匹配换行符)。
Pattern方法
split方法
Pattern提供了一个split方法，可以按照给定的正则表达式将字符串进行分割：
```java
String splitRegex = "\\([A-D]\\)";
String splitInput = "(A)非常满意  (B)满意   (C)一般  (D)不满意";
Pattern splitPattern  = Pattern.compile(splitRegex);
for(String str : splitPattern.split(splitInput)){
	System.out.println(str);
}
```
运行结果：
```bash
非常满意  
满意   
一般  
不满意
```
# quote方法
有点儿类似于Pattern.LITERAL的功能，方法可以为给定的字符串创建LITERAL模式的正则表达式：
```java
String quote = Pattern.quote("\\d");
System.out.println(Pattern.matches(quote, "\\d"));//true
System.out.println(Pattern.matches("\\d", "\\d"));//false
```

String类也提供了一些支持正则表达式的方法：
# matches方法：
```java
String str = "1234";
System.out.println(str.matches("(\\d)+"));//true
```

字符串的split方法和replace方法也支持正则表达式：
```java
String str2 = "a.1分       b.2分      c.3分       d.4分";
for(String score : str2.split("[a-d]+\\.")){
	System.out.println(score);
}

String str2 = "a.1分       b.2分      c.3分       d.4分";
str2 = str2.replaceAll("([a-d])+\\.", "[$1]");
System.out.println(str2);
```
运行结果：
```bash
[a]1分       [b]2分      [c]3分       [d]4分
```
这里用到了 反向引用，$1表示的就是匹配出来的[a-d]。 


# Matcher方法：
## lookingAt方法：
```java
String str3 = "foooooooooooo";
Matcher matcher3 = Pattern.compile("foo").matcher(str3);
System.out.println(matcher3.lookingAt());//true
System.out.println(matcher3.matches());//false
```
字符串是否包含所指定的正则表达式，不用通过find方法挨个找，lookingAt可以一次性实现这个功能。

## 字符串替换：
```java
double d = 1.0 / 3;
String numberRegex = "(\\d+\\.\\d{2,3})\\d*";
Matcher matcher4 = Pattern.compile(numberRegex).matcher("one divide by three eq : " + d +"");
String result = matcher4.replaceFirst("$1");
System.out.println(result);// OUPUT: one divide by three eq : 0.333
```
以上代码实现讲字符串保留小数的功能（只有两位小数位则不动）。

