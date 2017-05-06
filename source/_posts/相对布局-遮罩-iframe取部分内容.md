---
title: 相对布局 遮罩 iframe取部分内容
date: 2011-07-05 22:02
categories: Web前端
tags: 
- HTML
---
```html
title: 相对布局 遮罩 iframe取部分内容 date: 2011-07-05 22:02 categories: tags:
- HTML
<html>
<body>

其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
<div style="border:1px solid #0000ff;position:relative">
<div style="border:0px solid red;display:inline;width:220px" >

  <iframe src="http://www.jiankangzhuli.com/blog/" id="aa" disabled="disabled" marginheight="200" frameborder="1" scrolling="no" border="1" style="width:680px; height:650px; margin:-365px 0  0 -300px;border:0px solid #ff00ff" scrolling="no"></iframe>
 
   <a href="http://www.jiankangzhuli.com/blog/" style="font-size:12px">进入微博</a>

</div>
<div style="width:380px;height:285px;display:inline;float:both;position:absolute;top:0px;left:0px;border:1px solid red; 
text-align:center; 
background-color:#DBDBDB; 
filter:Alpha(opacity=0); 
opacity:0.0;
z-index:100000;
cursor:pointer"
onclick="window.location='http://www.jiankangzhuli.com/blog/';"
title="点击进入微博"
>
  &nbsp;
</div>
</div>

其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>
其他内容<br>

</body>
</html>
```