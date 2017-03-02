---
title: 使用Gradle快速生成项目升级文件
date: 2017-03-02 13:52:18
tags:
- Gradle
- 持续集成
categories: 技术浮云
---
# 安装Gradle
参考这里:[https://gradle.org/install](https://gradle.org/install)。

# 打包脚本
在Tomcat下创建一个build.gradle文件：
```bash
task zipRoot(type:Zip) {
 archiveName "Update-file.zip"
 from ('webapps')
 exclude(
 'docs',
 'examples',
 'host-manager',
 'manager',
 '**/lib',
 '**/oss.properties',
 '**/log4j.properties',
 '**/jdbc.properties',
 '**/rds.properties',
 '**/auth2-config.properties'
 )
}
```
做升级时，我一般只复制经常修改的文件，项有些配置文件是不能覆盖的（生产环境和开发环境是不一样的）。

# 编写DOS脚本
在build.gradle文件的相同位置创建zipRoot.bat脚本，内容如下：
```bash
gradle zipRoot
```

这样，每次要生成升级文件时，只需双击zipRoot.bat文件就可以自动生成文件了。