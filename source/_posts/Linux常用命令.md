---
title: Linux常用命令
date: 2011-04-30 14:33
categories: 
- Linux
tags: 
- Linux
- Java
- CSS
---

解压zip文件: 
```bash
[root@zdfl webapps]# unzip crm.zip
[root@zdfl webapps]# unzip -d crm crm.zip
```

删除文件或目录: 
```bash
[root@zdfl webapps]# rm -rf chart
[root@zdfl webapps]# rm -rf css
[root@zdfl webapps]# rm -rf fileUpload
[root@zdfl webapps]# rm -rf images
[root@zdfl webapps]# rm -rf install
```

创建目录: 
```bash
[root@zdfl webapps]# mkdir crm
```
vi编辑器： 
进入编辑状态 
vi 文件名 
进入字符插入状态： 
i 
Esc,:进入命令输入模式 
wq保存并推出 
q!退出不保存