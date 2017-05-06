---
title: 由request.getSession()想到的函数参数设计原则
date: 2014-04-16 18:01
categories: Java
tags: 
---
我们经常这么干，通过request.getSession()获取到session，然后查询其中的变量来判断当前用户是否登录。一天遇到有人问，能不能通过request.getSession()==null来判断用户没有登录呢？立马想到，什么时候会返回空呢？浏览器每次访问到服务器时，服务器会自动给创建一个session啊。那不能为空啊?但是超时或者通过手动的invalidate()之后session确实会失效，失效了返回就应该是空啊?

查看API才知道getSession()还有另外一个可以传递boolean类型参数的版本，传递true表示没有获取到session时自动创建一个，传递false则不创建。疑问解开了，还是自己太二了，不了解人家的API。但是后来发现，网上其他很多人都遇到了这个相同的疑问，开始反思这API设计的是否有问题。之所以出现这样的疑问，是因为大家很容易将这个函数的boolean参数给遗忘掉，以至于到最后大家都每天熟练的使用request.getSession()，只知道他能获取到session。这是函数设计上的一个忌讳，《Clear Code》中说**标识参数（布尔参数）的存在意味着你在公然宣布，这个函数做了不止一件事**，即便是HttpServletRequest中的getSession做了重载，但是它起到的效果是一样的（对外界来说是一个函数做了两件事情）。

另外，getSession()函数的名称和其实际的操作内容也有出入，《Clear Code》中说**函数实际的操作要与名称相符，不能背后还有其他副作用（这也是“只做一件事”的原则）**，而getSession()却不仅仅只是“get“，还带有了”create“的含义。

当然了，也许设计者考虑的是，getSession(true)比getSession(false)更为常用，并且创建一个session也无关紧要，也不影响谁，所以有意将getSession(flase)给弱化了。