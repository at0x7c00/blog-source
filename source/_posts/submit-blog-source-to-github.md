---
title: 提交博客源码到Github中
date: 2017/01/09 18:08
tags: hexo
categories: 折腾
---


成功将博客迁移到Github Pages后，我可以用Github来托管我的学习笔记了。每次用MarkdownPad将笔记整理好，打好标签，写好分类，使用Hexo生产静态页面，然后提交到Github Pages就行了。

现在要处理的一个问题是，如何将这些Markdown文件也提交到GitHub中。这样，我就可以在任意一台机器上开始开始记笔记了。

提交源码的时候会遇到的一个问题是，包含.git文件夹的文件夹，最终在github上会显示成灰色的不可用文件夹。我们需要做的就是把想要提交到github中去的文件夹下的.git文件夹删掉。

物理上删除.git文件夹之后，还需要让Git发现这个文件夹。比如我们将theme/next中的.git文件夹删除之后，需要执行以下命令来更新Git的状态：

```bash
$ git rm --cached themes/next
$ git add themes/.
```
其他带有.git文件夹的文件夹类似。

***

这样，在另外一台电脑上就可直接检出博客源码了：

```bash
$ git clone git@github.com:<your git userid>@<your git projectid>
```

得到类似如下的内容：

```bash
-blog-source/
  -.git/
  -package.json
  -scaffolds/
  -source/
  -themes/
  -_config.yml
```

在blog-source中执行如下代码来初始化Hexo:

先安装Hexo:
```bash
$ npm install hexo --save
```


安装完成之后的目录结构大致如下：

```bash
-blog-source/
  -.git/
  -.npmignore
  -node_modules/
  -package.json
  -scaffolds/
  -source/
  -themes/
  -_config.yml
```

将source/_posts/、.git、_config.yml、db.json和pachage.json备份到其他文件夹。

>hexo init 命令会导致.git文件夹被删除、_config.yml文件被覆盖、srouce/_posts/hello-world.md文件被覆盖。所以需要备份以便还原。

执行hexo的初始化

```bash
$ hexo init
```

再将上述备份的文件复制回来,最终得到如下文件：

```bash
-blog-source/
  -.git/
  -.npmignore
  -db.json
  -node_modules/
  -package.json
  -public/
  -scaffolds/
  -source/
  -themes/
  -_config.yml
```


生成并测试：

```bash
$ hexo generate
$ hexo server
```

部署:

```bash
$ hexo deploy
```

大功告成~