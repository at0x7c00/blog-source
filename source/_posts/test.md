---
title: JavaScript的自执行方法和模块模式
date: 2016/12/22 20:46:25
tags: Javascript
categories: 技术浮云
---


首先，每个方法定义的局部变量，在外部是无法访问的，比如：
```javascript
function foo(){
    var name;
}
```
这里的name变量对外界没有任何的污染，不会影响到其他地方的代码。如果每个模块的代码都这样写，那么就都不会打架了。

利用的函数的这一特性来实现隔离非常有效。通常情况下，我们只需要一个匿名的自执行函数就可以了：
```javascript
(function(){
   var name = 'Tom';
   return name;
})()
```
如果仅仅是return一个name那大可没必要这么干，我们需要返回一个具有类似JavaBean读写功能的对象：
```javascript
(function(){
   var name = 'Tom';
   var age = 10;
   return {
        name :name,age:age
   }
})()
```
这里返回了一个JSON对象，既然是对象，那么完全可以是有方法的：
```javascript
(function(){
   var name = 'Tom';
   var age = 10;
   return {
        name:name,age:age,
        setName(n){
            name = n;
        },
        setAge(a){
            age = a;
        }
   }
})()
```
是的，不光可以有方法，方法中竟然还能访问早已经执行完毕了的那个自执行方法的变量。真的很神奇。

神奇的背后其实是闭包在起作用。自执行方法虽然早在setName和setAge方法之前就已经执行完毕了，但因为后两个方法的定义中引用到了自执行方法中的变量。那么JavaScript引擎在为后两个函数创建闭包的时候会将自执行方法的变量“保留”。

这样就实现了代码的隔离（避免带来污染）还能合理访问的目的。