---
title: JQuery插件开发相关
date: 2014-03-10 22:21
categories: 
tags: 
---

翻看大多数js工具源码时经常遇到这样的代码：
代码1:
```javascript
（function($){
    // some code here...
}）(JQuery);
```

今天仔细琢磨了一下，这个代码实际的功能就是 定义了一个匿名函数，然后在页面load完毕之后执行了这个函数。它的简化版是：
代码2：
```javascript
(function(){
//some code here...
})();
```

这说明代码1中其实是将JQuery这个对象作为参数传递给了匿名函数，而匿名函数中给参数（形参）起了个另外的名字$。

记录一下今天在实现一个 将“您有新消息”内容在页面中闪烁功能的过程：
这个功能如果简单实现起来，只需要像下面这样就可以了：
```javascript
window.setInterval(function(){
      $("#targetId").toggle();
},500);
```

只需要这样，这个id为targetId的元素就开始在页面中闪烁了。但是，我希望我能很方便的让其他页面元素也能很轻松的实现这样的功能，并且我还能随时控制说让它不要再闪烁了。我需要实现一个像jqueryUI中$("#modal-dialog"). dialog();这样的工具。
这需要扩展jquery的函数了，借助上面的原理，有了下面的代码：
```javascript
 (function($){
	$.fn.extend({
		flash:function(opt){
			$(this).each(function(){
				var tmp = $(this);
				window.setInterval(function(){
					$(tmp).toggle();
				},500);
			});
		}
	});
})(Jquery);
```


这里，我用flash来作为这个闪烁功能的函数名，opt是传递的参数。但是，这个样子，这个方法只能调用一次，再多调用的话，元素的闪烁会加快甚至失控，因为每次都直接toggle了。所以我需要有一个数组来记录这些元素的interval对象。
```javascript
 (function($){
<span style="white-space:pre">	</span>var _intervalArray = [];
	$.fn.extend({
		flash:function(opt){
			$(this).each(function(){
				var tmp = $(this);	
 				var targetId = tmp .attr("id");
 				var interval = _intervalArray[targetId];
				if(!interval){
					interval = window.setInterval(function(){
						$(tmp).toggle();
					},500);
					_intervalArray[targetId] = interval;
				}
			});
		}
	});
})(Jquery);
```

js是一个很奇怪的语言，字符串也能作为数组数组访问index。这里我通过一个数组来记录它，在下一次再调用的时候，如果发现已经在闪烁了就不要再interval了。 
我希望通过$("SELECTOR-CODE").flash();来启动闪烁，通过$("SELECTOR-CODE").flash("stop");来停止闪烁，具体如下：
```javascript
(function($){
 	var _intervalArray = [];
 	$.flashOnTarget = function flashOnTarget(targetId){
 		$("#"+targetId).toggle();
 	};
 	$.fn.extend({
 		flash:function(opt){
 			var target = $(this);
 			for(var i=0; i<target.length; i++ ){
 				var targetId = $(target[i]).attr("id");
 				var interval = _intervalArray[targetId];
 				if(opt=='stop' && interval){
 					window.clearInterval(interval);
					 $(target[i]).show();
 				}else{
 					if(!interval){
 						interval = window.setInterval("$.flashOnTarget('"+targetId+"')",300);
 						_intervalArray[targetId] = interval;
 					}
 				}
 			}
 			return target;
 		}
 	});
})(jQuery);
```


最后这个代码中，我把setInterval中的匿名函数单独独立出来了，为了能够把代码都统一放置在 
(function($){

})(); 
中，我把这个函数定义成了一个jquery的一个扩展函数。这里还有一个有趣并值得注意的地方，window的setInterval和setTimeout两个函数是 没办法传递参数的，为了变相的传递参数，只能把它写成一个字符串了。 
下面是完整代码：
```javascript
<html>
<head>
<title></title>
<script src="http://code.jquery.com/jquery-1.11.0.min.js"></script>
<script type="text/javascript">
$(function(){
	$("span").flash({cycle:100});
	window.setTimeout(function(){
		$("#sp1,#sp2").flash("stop");
	},5000);
});
(function($){
	var _intervalArray = [];
	$.flashOnTarget = function flashOnTarget(targetId){
		$("#"+targetId).toggle();
	};
	$.fn.extend({
		flash:function(opt){
			var target = $(this);
			for(var i=0; i<target.length; i++ ){
				var targetId = $(target[i]).attr("id");
				var interval = _intervalArray[targetId];
				if(opt=='stop' && interval){
					window.clearInterval(interval);
					$(target[i]).show();
				}else{
					if(!interval){
						interval = window.setInterval("$.flashOnTarget('"+targetId+"')",opt? (opt.cycle ? opt.cycle:400) : 400);
						_intervalArray[targetId] = interval;
					}
				}
			}
			return target;
		}
	});
})(jQuery);
</script>
<style>
span{
 border:1px solid red;
 color:red;
 padding:5px;
}
</style>
</head>
<body>
<span id="sp1">您有新消息1</span><br><br>
<span id="sp2">您有新消息2</span><br><br>
<span id="sp3">您有新消息3</span><br><br>
<span id="sp4">您有新消息4</span><br><br>
</body>
</html>
```










