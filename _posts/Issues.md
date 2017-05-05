---
title: 经典 Issues 集锦
date: 2017-03-15 15:15:15
categories: 
- 杀虫
- Java
- 待完善
tags: 
- bug
- 调试
- 错误
- 异常
- Issues
---

什么是经验？经验，就是遇到问题之后，你通过努力把它解决了，这就是你的经验！
- Java bugs
- MySQL bugs
- HTML bugs

<!-- more -->

在程序员的道路上，会遇到各种各样的问题和错误，我认为我不可能记住每一个问题的解决方式，好脑子不如烂笔头嘛，所以我要把我今后遇到的各种问题与错误都记录在这里～
## Java Issues
### An API baseline has not been set for the current workspace.	

Windows → Preferences → Plug-in Development → API Baselines
在 Options 里找到  Missing API baseline ，根据自己的情况改成 Warning 或者 Ignore，点击 Apply 应用即可。

>建议在[这里](http://www.ibm.com/developerworks/library/os-eclipse-api-tools/)研读一下 API baselines ，然后再决定这些 API baselines 是否对你有用。

<br/>
### java.lang.ClassNotFoundException: com.mysql.jdbc.Driver
**报错信息：**
java.lang.ClassNotFoundException: com.mysql.jdbc.Driver...

**报错原因：**
1. 驱动类名称写错
2. 没有添加 jar 包
	1. 项目中没有添加 mysql-connector-java-x.x.xx-bin.jar 包
	2. Web 项目中添加了 mysql-connector-java-x.x.xx-bin.jar 包，但是在 Tomcat 安装目录的 lib 目录下没有该 jar 包

**解决方法:**
- 对于第一种原因仔细查找下改掉拼错的单词就行了，不多解释啦。
- 对于第二种报错原因的第一种情况，下载 mysql-connector-java-x.x.xx-bin.jar 添加进去就可以了，这里也不多说。
- 悲催的我遇到了另一种情况，已经将 mysql-connector-java-x.x.xx-bin.jar 包添加到项目中，测试数据库连接也是没有问题的，但是在 server 中运行时却报错...此时只要将相同的 jar 包复制一份到 Tomcat 安装目录中的 lib 文件夹中重启 Tomcat 服务器即可。

<br/>
### ERROR StatusLogger No log4j2 configuration file found.
**报错信息：**
ERROR StatusLogger No log4j2 configuration file found. Using default configuration: logging only errors to the console. Set system property 'org.apache.logging.log4j.simplelog.StatusLogger.level' to TRACE to show Log4j2 internal initialization logging.

**报错原因：**
- 没有找到 log4j2 的配置文件

**解决办法：**
因为用的是 log4j2 所以不能按照大多数网站上写的配置 `log4j.properties` ，需要将配置文件名改为 `log4j2.properties`。

**参考：**


<br/>
### <!DOCTYPE log4j:configuration SYSTEM "log4j.dtd"> 系统找不到指定的文件
**警告信息：**
The file cannot be validated as the XML definition "项目路径\src\log4j.dtd (系统找不到指定的文件。)" that is specified as describing the syntax of the file cannot be located.

**报错原因：**
- 要么缺少 log4j.dtd 文件
- log4j.dtd 名称写错。
- log4j.dtd 文件未放在 src 目录下

**解决办法：**
- log4j.dtd 位置（若已有 log4j.dtd 文件则忽略此条）
	- log4j 1.x：使用解压缩工具解压log4j.jar文件，找到org/apache/log4j/xml目录下的log4j.dtd 文件。
	- log4j 2.x：使用解压缩工具解压 log4j-core-2.8.2.jar 文件，找到根目录下的 Log4j-events.dtd 文件。
- 将找到的 log4j.dtd 文件拷贝到项目路径的 src 目录下即可。

**参考：**
[log4j.xml 提示不能找到 log4j.dtd](http://blog.csdn.net/linshutao/article/details/6578788)

<br/>
## MySQL Issues
### #1089-incorrect prefix key
```MySQL
CREATE TABLE `table`.`users` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(50) NOT NULL,
    `password` VARCHAR(50) NOT NULL,
    `dir` VARCHAR(100) NOT NULL,
    PRIMARY KEY (`id`(11))
) ENGINE = MyISAM;
```
**报错信息：**#1089-incorrect prefix key;the used key part isn't a string,the used length is longer than the key part,or the storage engine doesn't support unique prefix keys.

**报错原因：**
- 在定义 PRIMARY KEY 时候使用了 `(id(11))`, 这定义了 prefix key —— 也就是主键前面的 11 个字符只能用来创建索引。prefix key 只支持 CHAR、VARCHAR、BINARY 和 VARBINARY 类型，而这里使用了 int 类型，所以报错了。翻译自 [stackoverflow](http://stackoverflow.com/questions/28932281/what-is-wrong-with-my-sql-here-1089-incorrect-prefix-key)。

**解决办法：**
- 将
```MySQL
PRIMARY KEY (`id`(11))
```
改为
```MySQL
PRIMARY KEY (`id`)
```

**参考：**[MySQL 官方文档](https://dev.mysql.com/doc/refman/5.5/en/create-index.html)

<br/>
### com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
**报错信息：**
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure.
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
**报错原因：**
- 未开启 mysql 服务

**解决办法：**
- 启动 mysql 服务

<br/>
### 使用 PreparedStatement 将中文数据存入 mysql 中显示 "?"
**表现形式：**
- 在数据库中查看插入的中文数据全都显示 "?"，Java 取出来的数据也显示 "?"
- 在数据库工具中手动插入的中文数据正常显示，Java 取出来的数据也正常显示。
- preparedStatement 语句执行前，通过
```Java
System.out.println(preparedStatement.toString());
```
输出的 mysql 语句，其中文数据也显示 ?
```
com.mysql.jdbc.JDBC42PreparedStatement@4102b8: INSERT INTO catgory VALUES ('d6509', '??', 1)
```

**问题原因：**
设置驱动时未指定编码格式。

**解决办法：**
将原先的数据库驱动配置
```
jdbc:mysql://localhost:3306/数据库名
```
改为
```
jdbc:mysql://localhost:3306/数据库名?characterEncoding=utf8
```
>其中数据库名为你自己的数据库名称

<br/>
## HTML Issues

<br/>
## Git
###
**错误提示：**You asked to pull from the remote 'master', but did not specify
a branch. Because this is not the default configured remote
for your current branch, you must specify a branch on the command line.

**报错原因：**你拉取指定名为 “master” 的远程仓库，但是该远程仓库没有指定分支。因为你当前的分支不是默认的远程仓库，所以必须要指定一个分支。

**解决办法：**
- 编辑项目下的 `.git/config` 文件，加入下面的代码：
```
[branch "master"]
  remote = origin
  merge = refs/heads/master
```

**参考：**[StackOverFlow](http://stackoverflow.com/questions/4847101/git-which-is-the-default-configured-remote-for-branch)