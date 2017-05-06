---
title: [转]在Linux下安装rarlinux
date: 2011-04-30 11:03
categories: 
- Linux
tags: 
- Linux
- VB
- F#
---

rarlinux 软件下载地址：http://www.rarsoft.com/download.htm 

到目前为止最新的版本为3.90 beta 3，这是共享版本。 

本文所用的Linux操作系统为： 
Red Hat Enterprise Linux AS release 4 (Nahant Update 4)，内核版本2.6.9-42.ELsmp，32位版本。 

选择RAR 3.90 beta 3 for Linux： 
$wget http://www.rarsoft.com/rar/rarlinux-3.9.b3.tar.gz 

如果是64位的系统，就用RAR 3.90 beta 3 for Linux x64。 

用root帐户安装： 
```bash
$su - 
#tar -zxvf rarlinux-3.9.b3.tar.gz 
#cd rar 
#make 
#make install 
#exit 
```
安装过程结束。 

运行rar --help可以看到帮助信息，如果出现下列信息： 
```bash
#rar: /lib/tls/libc.so.6: version GLIBC_2.4' not found (required by rar) <br>#rar: /lib/tls/libc.so.6: versionGLIBC2.7' not found (required by rar) 
```
则执行： 
```bash
#cp -f rarstatic /usr/local/bin/rar 
```
这样就可以使用rar命令了。 

----

rar命令用法： 

解压文件： 
把压缩包的内容解压到当前目录 
```bash
$rar e File.rar 
```
把压缩包的内容解压到指定目录，比如/tmp/下面
```bash 
$rar e File.rar /tmp/ 
```
把压缩包解的内容压到指定目录，比如/tmp/下面，包含压缩包中的路径
```bash 
$rar x File.rar /tmp/ 
```
----

压缩: 

以默认压缩率压缩指定的一个文档，比如SourceFile 
```bash
$rar a File.rar SourceFile 
```

以最大压缩率压缩指定的一个文档，比如SourceFile 
```bash
$rar a -m5 File.rar SourceFile
``` 

压缩指定的一个目录下的所有文档，比如/Direcroty目录下的所有文档 
```bash
$rar a File.rar /Direcroty/ #目录可以是相对目录，也可以是绝对目录 
```

压缩指定的一个目录下的任何文档，比如/Direcroty目录下的所有文档和所有子目录
```bash 
$rar a -r File.rar /Direcroty/ 
```

压缩指定的一个目录下的所有文档，比如/Direcroty目录下的所有文档和所有子目录，但是不包含空目录 
```bash
$rar a -r -ed File.rar /Direcroty/ 
```
压缩指定的一个目录下的所有文档，比如当前目录下的Direcroty目录，连目录也一起压缩，包括子目录
```bash 
$rar a File.rar Direcroty/ 
```
---- 

查看压缩包中的文档： 
```bash
$rar l File.rar   或   $rar v File.rar 
```
查看压缩包中的文档(只看有什么文档)
```bash 
$rar lb File.rar 或 $rar vb File.rar 
```
查看压缩包中的文档(详细信息)
```bash 
$rar lt File.rar 或 $rar vt File.rar 
``` 
