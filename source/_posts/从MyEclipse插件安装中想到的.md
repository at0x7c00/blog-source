---
title: 从MyEclipse插件安装中想到的
date: 2014-12-02 23:03
categories: 日记
tags: 
---
title: 从MyEclipse插件安装中想到的 date: 2014-12-02 23:03 categories:
tags:
网上一搜“Eclipse插件安装”一大堆文章，之前也懒得记。刚才又安装了以下SVN插件，伤心啊。
我已经将SVN插件压缩文件下载到了本地，然后使用help->Install from site的安装方式来安装，结果先滚出一堆什么乱七八糟的东西来，好像是在检查网络上的一个路径，TM，我不都已经下载下来了吗？你检查什么呢？！
好不容易经过了这一步，然后我选择了我下载的压缩文件，然后就开始等待了。。。
![](http://img.blog.csdn.net/20141202222714375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


一直停在这个画面，并且每有打算收场的意思，网上一搜，又是因为天朝的威望导致的，无语。
没辙了，换成dropin方式安装吧，刚开始我的安装文件解压后在了这个地方：
![](http://img.blog.csdn.net/20141202223040219?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

然后，dropin文件夹下如网上所述创建了link文件：
```bash
path=D:\\tool\\MyEclipse2014\\add\\Subversive-2.0.0.I20140609-1700
```

重启之后，发现MyEclipse没有任何变化，忽然想起之前好像说过解压的安装文件要放在eclipse文件夹下，于是调整如下：
![](http://img.blog.csdn.net/20141202223423861?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

重启MyEclipse，果然见证了奇迹。开心啊，终于如愿了一把，准备记录一下心得，以后一定要记得这个。还没完呢，接下来要安装SVN connector吧，有了之前的经验，我和安装SVN一样把svn connector的安装文件解压到了eclipse文件夹下，然后下了一个link文件指向eclipse文件夹所在文件夹：
```bash
path=D:\\tool\\MyEclipse2014\\add\\eclipse\\Subversive-connectors
```

满心自行重启MyEclipse，结果啥都没安装上，哎，思前想后，实在没辙，算了把eclipse文件夹去掉吧：
![](http://img.blog.csdn.net/20141202223959153?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


同理修改link文件内容，重启MyEclipse，安装成功！心里啊，悲喜交加！

想起之前老婆问的一个问题，为啥从网页上复制的东西粘贴到Excel文件里会带有链接？还有为啥粘贴的数字编号会自动使用了科学计数方式显示。我说你不想这样，就右击，然后选择粘贴位普通字符啊，不行就先将单元格格式设置位文本，她说这样多麻烦啊。我说，问题产生的原理就在这里，明白原理了，就不要遇到问题了一直堵在这里往前撞，应该绕开它。我们总是在写程序的时候，为某些理所当然应该没问题但却又产生了问题的情况气愤不已，结果到头来不是发现自己犯了低级错误就是孤陋寡闻，或者是别人的一个错误，让你在那儿头疼了半天。实在不值。

最近CSDN的博客系统老是响应缓慢，我想是不是被那个闲的蛋疼的大侠给黑了？

让人头痛的IE，每次想起就想骂骂微软，你不会写浏览器就别在那儿瞎整！你拉完屎了，大家跟着你臭！

Chrome最近升级的debug模式下的页面测量功能，有的时候本来想把界面拉大，结果反而变小了。有点儿像Dreamware，就是不知道怎么把界面拉大，偶然在尝试把界面拉小的时候结果界面却变大了，彻底顿悟啊！终于知道Chrome背后的呵呵呵了：你猜猜看，到底怎样才能将界面拉大？

每当遇到这种情况，就觉得这软件不咋地。写程序多了，就会对软件的bug越敏感（即使自己写的软件也bug百出），遇到网络上有用户登录或这次页面连验证码都不带的，就想黑他一把：这东西也敢放在这里？经常因为这样的一些软件而感到有些气愤，干嘛做得这么差劲？
