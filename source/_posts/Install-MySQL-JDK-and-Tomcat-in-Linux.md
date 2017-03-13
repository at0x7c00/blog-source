---
title: 'Install MySQL,JDK and Tomcat in Linux'
date: 2017-01-04 17:13:23
categories: 技术浮云
tags: 
- MySQL
- Linux
- Java
- Tomcat
---

# 安装MySQL

在官网[http://dev.mysql.com/downloads/mysql#downloads](http://dev.mysql.com/downloads/mysql#downloads)选择合适您操作系统的版本进行下载， 这里以选择Linux Generic为例。

>MySQL官网上只能下载到最新的版本，想要下载以前的版本找起来非常费劲。你可以从这里来下载:
>[http://download.softagency.net/MySQL/Downloads/MySQL-5.5/](http://download.softagency.net/MySQL/Downloads/MySQL-5.5/)

把下载的文件上传到Linux某个临时文件夹里，然后解压文件，得到一个文件夹：

```bash
tar -zxvf mysql-5.5.45-linux2.6-x86_64.tar.gz
```

```bash
drwxr-xr-x 13 root root      4096 Jan  3 20:09 mysql-5.5.53-linux2.6-x86_64
-rw-r--r--  1 root root 185923648 Jan  3 20:00 mysql-5.5.53-linux2.6-x86_64.tar.gz
```

复制解压后的mysql目录到系统的本地软件目录:
执行命令：cp mysql-5.5.45-linux2.6-x86_64 /usr/local/mysql -r

添加mysql用户和mysql用户组:
```bash
groupadd mysql
useradd -r -g mysql mysql
```

切换到/user/local/mysql目录下，改变mysql目录的用户和组：

```bash
cd /usr/local/mysql/
chown -R mysql:mysql ./
```

## 安装数据库

```bash
./scripts/mysql_install_db  --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/mysql.pid --tmpdir=/tmp
```

>如果报cannot open shared object file错误，可以执行如下命令：
>apt-get install libaio1 libaio-dev


把mysql文件夹的所属组和所属用户修改为root：
```bash
chown -R root:root ./
```
把data文件夹的所属组合所属用户修改为mysql：
```bash
chown -R mysql:mysql data
```
最终效果如下：

```bash
[root@iZ2zegp7tdd3qrlnkeswi3Z install]# cd /usr/local/
[root@iZ2zegp7tdd3qrlnkeswi3Z local]# ll
total 60
drwxr-xr-x   6 root root 4096 Dec 30 17:36 aegis
drwxr-xr-x.  2 root root 4096 Jan  3 20:44 bin
drwxr-xr-x.  2 root root 4096 Sep 23  2011 etc
drwxr-xr-x.  2 root root 4096 Sep 23  2011 games
drwxr-xr-x.  2 root root 4096 Sep 23  2011 include
drwxr-xr-x.  2 root root 4096 Sep 23  2011 lib
drwxr-xr-x.  2 root root 4096 Sep 23  2011 lib64
drwxr-xr-x.  2 root root 4096 Sep 23  2011 libexec
drwxr-xr-x  13 root root 4096 Jan  3 20:11 mysql
drwxr-xr-x.  2 root root 4096 Sep 23  2011 sbin
drwxr-xr-x.  5 root root 4096 Aug 14  2014 share
drwxr-xr-x.  2 root root 4096 Sep 23  2011 src
[root@iZ2zegp7tdd3qrlnkeswi3Z local]# cd mysql
[root@iZ2zegp7tdd3qrlnkeswi3Z mysql]# ll
total 72
drwxr-xr-x  2 root  root   4096 Jan  3 20:12 bin
-rw-r--r--  1 root  root  17987 Jan  3 20:11 COPYING
drwxr-xr-x  6 mysql mysql  4096 Jan  4 13:56 data
drwxr-xr-x  2 root  root   4096 Jan  3 20:11 docs
drwxr-xr-x  3 root  root   4096 Jan  3 20:11 include
-rw-r--r--  1 root  root    301 Jan  3 20:11 INSTALL-BINARY
drwxr-xr-x  3 root  root   4096 Jan  3 20:11 lib
drwxr-xr-x  4 root  root   4096 Jan  3 20:11 man
drwxr-xr-x 10 root  root   4096 Jan  3 20:11 mysql-test
-rw-r--r--  1 root  root   2496 Jan  3 20:11 README
drwxr-xr-x  2 root  root   4096 Jan  3 20:11 scripts
drwxr-xr-x 27 root  root   4096 Jan  3 20:11 share
drwxr-xr-x  4 root  root   4096 Jan  3 20:11 sql-bench
drwxr-xr-x  2 root  root   4096 Jan  3 20:11 support-files
```

## 添加开机启动

执行命令把启动脚本放到开机初始化目录：
```bash
cp support-files/mysql.server /etc/init.d/mysql
```

启动之前，需要修改mysql的编码为utf8：
1.找到mysql的配置文件，拷贝到etc目录下，把/usr/local/mysql/support-files/my-medium.cnf 复制到 /etc/my.cnf。即用命令：
```bash
cp /usr/local/mysql/support-files/my-medium.cnf  /etc/my.cnf
```
2.打开my.cnf修改编码
在[mysqld]下增加

```bash
character-set-server=utf8
```

启动mysql服务：执行命令
```bash
service mysql start
```

修改MySQL的root用户密码：
```bash
./bin/mysqladmin -u root password 'passw0rd'
```

把mysql客户端放到默认路径：
```bash
ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
```

然后测试是否能顺利登录MySQL：
```bash
mysql -uroot -p[your password]
```

记得给MySQL添加开机启动：

```bash
cd /etc/init.d/
chkconfig --add mysql
```
## 设置MySQL不区分大小写
修改MySQL的配置文件my.cnf，在[mysqld]部分添加如下配置选项：
```bash
lower_case_table_names = 1
```
这是没有办法的办法，最好的办法还是应该严格按照规范执行。目前觉得最佳实践是SQL语句中的库名、表名、字段名等都用小写就没这些问题了。

## 定时备份任务
[参考这里](http://blog.huqiao.me/2017/01/05/MySQL-Backup/)。

# 安装JDK

到官网下载合适的安装包，下载地址：[http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk7-downloads-1880260.html](http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk7-downloads-1880260.html).

>和MySQL一样，因为JDK也是Oracle公司的，在官网也只提供最新版本的下载。我这里备份了一个：[http://pan.baidu.com/s/1qYextPq](http://pan.baidu.com/s/1qYextPq)。

JDK的安装很简单，只需要将文件解压到/usr/local/中即可：

```bash
[root@iZ2zegp7tdd3qrlnkeswi3Z local]# ll
total 60
drwxr-xr-x   6 root root 4096 Dec 30 17:36 aegis
drwxr-xr-x.  2 root root 4096 Jan  3 20:44 bin
drwxr-xr-x.  2 root root 4096 Sep 23  2011 etc
drwxr-xr-x.  2 root root 4096 Sep 23  2011 games
drwxr-xr-x.  2 root root 4096 Sep 23  2011 include
drwxr-xr-x   8 root root 4096 Jan  4 11:56 jdk1.7.0_79
drwxr-xr-x.  2 root root 4096 Sep 23  2011 lib
drwxr-xr-x.  2 root root 4096 Sep 23  2011 lib64
drwxr-xr-x.  2 root root 4096 Sep 23  2011 libexec
drwxr-xr-x  13 root root 4096 Jan  3 20:11 mysql
drwxr-xr-x.  2 root root 4096 Sep 23  2011 sbin
drwxr-xr-x.  5 root root 4096 Aug 14  2014 share
drwxr-xr-x.  2 root root 4096 Sep 23  2011 src
```
最要的是设置环境变量。
本编辑器gedit（如果没安装可以用vi）打开/etc/profile，在文件最后添加
```bash
export JAVA_HOME=/usr/local/jdk1.7.0_79
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
保存后重新编译/etc/profile文件，测试Java是否可用:

```bash
[root@iZ2zegp7tdd3qrlnkeswi3Z local]# java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode) 
```


# 安装Tomcat

Tomcat的下载就不用多说了，可以在[官网](http://tomcat.apache.org/)找到下载链接。也可以使用这个镜像地址：[http://archive.apache.org/dist/tomcat/](http://archive.apache.org/dist/tomcat/)。

也和MySQL、JDK一样，解压到local目录中即可：
```bash
drwxr-xr-x   6 root root 4096 Dec 30 17:36 aegis
drwxr-xr-x.  2 root root 4096 Jan  3 20:44 bin
drwxr-xr-x.  2 root root 4096 Sep 23  2011 etc
drwxr-xr-x.  2 root root 4096 Sep 23  2011 games
drwxr-xr-x.  2 root root 4096 Sep 23  2011 include
drwxr-xr-x   8 root root 4096 Jan  4 11:56 jdk1.7.0_79
drwxr-xr-x.  2 root root 4096 Sep 23  2011 lib
drwxr-xr-x.  2 root root 4096 Sep 23  2011 lib64
drwxr-xr-x.  2 root root 4096 Sep 23  2011 libexec
drwxr-xr-x  13 root root 4096 Jan  3 20:11 mysql
drwxr-xr-x  10 root root 4096 Jan  4 16:43 NitsmUpload
drwxr-xr-x.  2 root root 4096 Sep 23  2011 sbin
drwxr-xr-x.  5 root root 4096 Aug 14  2014 share
drwxr-xr-x.  2 root root 4096 Sep 23  2011 src
drwxr-xr-x  10 root root 4096 Jan  4 13:50 tomcat-8.0.36
```
## 为Tomcat添加开机启动项
进入tomcat-8.0.36/bin目录中；
修改catalina.sh文件，在export处增加如下内容：
```bash
CATALINA_HOME=/usr/local/tomcat-8.0.36
JAVA_HOME=/usr/local/jdk1.7.0_79
```
创建catalina.sh链接到/etc/init.d中：
```bash
ln -s /usr/local/apache-tomcat-7.0.63/bin/catalina.sh  /etc/init.d/tomcat
```
添加到自动服务
```bash
update-rc.d  –f  tomcat  defaults（删除 defaults 换成remove）
```

## 使用chkconfig添加开机启动

在/etc/init.d/下创建tomcat文件，内容为：
```bash
#!/bin/bash
# description: Tomcat Start Stop Restart  
# processname: tomcat  
# chkconfig: 234 20 80  
JAVA_HOME=/usr/local/java/jdk1.7.0_79  
export JAVA_HOME  
PATH=$JAVA_HOME/bin:$PATH  
export PATH  
CATALINA_HOME=/usr/apache-tomcat-8.0.36  
  
case $1 in  
start)  
sh $CATALINA_HOME/bin/startup.sh  
;;   
stop)     
sh $CATALINA_HOME/bin/shutdown.sh  
;;   
restart)  
sh $CATALINA_HOME/bin/shutdown.sh  
sh $CATALINA_HOME/bin/startup.sh  
;;   
esac      
exit 0  
```

执行如下命令：
```bash
chkconfig –add tomcat
chkconfig –level 2345 tomcat on
```

## 修改Tomcat默认编码为UTF8

修改/usr/local/apache-tomcat-8.0.36/conf/server.xml文件：
```xml

    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8"/>

```

## 设置内存
要添加在tomcat 的bin 下catalina.sh 里，位置cygwin=false前 。

```bash
JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m" 
cygwin=false
```

> Tomcat8容易遇到启动缓慢的问题，参考[Tomcat启动缓慢问题解决](http://nobodyiam.com/2016/06/07/tomcat-startup-slow/)。
