---
title: 成功移居GitHub Pages
date: 2016-12-01
categories:	joys
tags: 
- GitHub
- 折腾
---
# GitHub+Hexo+Next
借助于这边文章：[《GitHub+Hexo+Next搭建免费独立个人博客》](http://www.jianshu.com/p/61987cec0fad)，终于在GitHub Pages中创建了自己的博客系统。生命在于折腾真的太对了。通过在这次折腾，发现了很多好玩的东西：
Vue.js、Markdown、Hexo、Nodejs、sublime text、React。感觉自己对于整个前端技术已经落后了好大一截啊，各种技术的堆砌，几乎已经看不到底层的脉络了。

用Git写博客的方式也是从没有想过的，线下使用MarkdownPad桌面编辑器来写博客，随时可以用git提交到GitHub中。这样的静态博客真的很方便。

其实平时的一些学习代码都可以整理成代码保存在GitHub中。接下来要好好研究研究Markdown了。

这个博客的主题[Next](http://theme-next.iissnan.com/)强调的是“精于心，简于形”，博客不应该有太多让人分心的地方，更多的是起到一个思考、记录和分享交流的目的。与其说坚持把这个博客写好，不如说应该让自己的生活应该多一些沉淀少一些浮躁。有了这个原则，写博客也就不用刻意地去坚持了。


# 二级域名配置
## CNAME内容
```text
blog.huqiao.me
```

## 阿里云域名配置
```text
记录类型 	主机记录 	解析线路 	记录值
CNAME	　　blog	　　默认	　　xooxle.github.io.
```

## dig记录
```bash
C:\Windows\System32>dig blog.huqiao.me +nostats +nocomments +nocmd
; <<>> DiG 9.10.1 <<>> blog.huqiao.me +nostats +nocomments +nocmd
;; global options: +cmd
;blog.huqiao.me.                        IN      A
blog.huqiao.me.         589     IN      CNAME   xooxle.github.io.
xooxle.github.io.       21844   IN      CNAME   github.map.fastly.net.
github.map.fastly.net.  10      IN      A       151.101.100.133
```

# 待解决问题

* 发布时间总是跟着最近一次提交时间变动
* 文章阅读次数
* 社交账号：微博、微信 