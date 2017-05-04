---
title: 使用crontab为MySQL创建定时备份任务
date: 2017-01-05 15:49:13
tags:
- MySQL
- corntab
categories: 数据库
---

选择一个文件夹，创建一个需要定时执行的脚步文件。

```bash
cd /usr/local/
mkdir mysql-backup
cd mysql-backup/
vi mysqlbackup.sh
```

在文件中输入如下内容：

```bash
#!/bin/sh
filename=$(date +"%y-%m-%d_%H%M%S")
/usr/local/mysql/bin/mysqldump -uroot -p[your password] [your db name] | gzip>/usr/local/mysql-backup/backupfiles/[your db name]-$filename.sql.gz
```
>注意：$(date +"%y-%m-%d_%H%M%S")中date和后面的+号一定要有空格，然后+号和后面的格式字符串一定不能有空格。也不知道是为什么。

这里其实还是使用了MySQL自带的mysqldump工具来进行的备份，只是多了一步压缩的操作。压缩的文件带上当前的系统时间。

改变文件属性，使其可以执行：

```bash
chmod 755 mysqlbackup.sh 
ll
total 4
-rwxr-xr-x 1 root root 170 Jan  5 16:03 mysqlbackup.sh 
```

接下来将这个可执行的文件配置到crontab中定时执行即可,执行crontab -e打开配置文件，输入以下内容：

```bash
1 1,12,18 * * * /usr/local/mysql-backup/mysqlbackup.sh
```
这里配置的是每天凌晨1点，中午12点和晚上6点分别执行一次备份。
可以执行service crond reload来重新载入crontab的配置。

配置项中各项的大概含义是：
```bash
*  *  *   *   *    command
|  |  |   |   |      需要执行的命令（可执行脚本路径）
|  |  |   |   |----->星期（0-7）
|  |  |   |--------->月份（1-12）
|  |  |------------->日（1-31）
|  |---------------->小时（0-23）
|------------------->分钟（0-59）
```
其中每一个*如果有多项时，可以用逗号分隔。
更多详细的crontab配置[参考这里](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)。

