---
title: mysql中文乱码
date: 2016-09-08 10:03:47
tags: 
- mysql
- 中文乱码
categories: Mysql
---
同事在部署服务器，导入数据库的时候出现中文乱码问题。网上Google了下，找到解决方法，在此记录下。

首先，登陆数据库，查看当前的字符集：
```
MariaDB>SHOW VARIABLES LIKE 'character%';
```
可以看到，默认的字符集为`latin1`.

找到Mysql的`my.conf`文件.一般默认在`/etc/mysql/my.cnf`  

在`[client]`,`[mysql]`等字段加入 `default-character-set=utf8`， 如下：  
```
[client]
port = 3306
socket = /var/lib/mysql/mysql.sock
default-character-set = utf8
```
而在`[mysqld]`字段加入 `character-set-server=utf8`

修改完成之后，重启mysql。
