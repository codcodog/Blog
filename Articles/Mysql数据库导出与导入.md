---
title: Mysql数据库导出与导入
date: 2016-08-22 23:20:40
tags: 
- mysql
categories: Mysql
---
在命令行下，如何对Mysql数据库进行数据的导出与导入？  

**导出数据**

在命令行输入：
```
mysqldump -uUSERNAME -pPASSWORD -hHOSTNAME USER_DATABASE > FILENAME.sql
```

**导入数据**
首先登陆数据库，
```
$ mysql -uUSERNAME -pPASSWORD
```
然后，导入数据：
```
mysql>source /PATH/FILENAME.sql
```
需要注意的是，`FILENAME.sql`文件路径不要填错。
