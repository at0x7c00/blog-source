---
title: [转]javascript获取鼠标位置
date: 2011-07-07 23:32
categories: 
- Javascript
tags: 
- prototype
---
由于IE和Firefox对鼠标当前位置获取方法不同(IE为event.x|y,FF为event.pageX|Y)，一般采用的是event.clientX代替两者，但当出现滚动条时event.clientX在IE和FF中的表现会略有不同。下面看看prototype和YUI如何处理这个问题... 

# prototype 

```javascript
  pointerX: 
function(event) {  
  return event.pageX || (event.clientX + (document.documentElement.scrollLeft || document.body.scrollLeft));  },  pointerY: function(event) {    return event.pageY || (event.clientY +      (document.documentElement.scrollTop || document.body.scrollTop));  },  
```

# YUI 
```javascript
getPageX: function(ev) {    
	var x = ev.pageX;    
	if (!x && 0 !== x) {        
		x = ev.clientX || 0;        
		if ( this.isIE ) {
			x += this._getScrollLeft();        
		}    
	}    
	return x;
},
getPageY: function(ev) {    
	var y = ev.pageY;    
	if (!y && 0 !== y) {        
		y = ev.clientY || 0;        
		if ( this.isIE ) {
			y += this._getScrollTop();        
		}    
	}    
	return y;
},
```
思路都一样，通过判断是否支持某对象来判断浏览器，然后增加Scrol的偏置值。 