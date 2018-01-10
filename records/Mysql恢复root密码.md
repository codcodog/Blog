---
title: Mysql恢复root密码
date: 2016-08-22 23:33:59
tags: 
- mysql
- password
categories: Mysql
---
有时候，在登陆数据库的时候，忘记了root用户的密码，但又不想重装，那么如何找回root密码呢？  

首先第一步，
> 停止数据库进程.

RedHat系列:
```
$ sudo /etc/init.d/mysqld stop
```
然后进入安全模式，
```
$ sudo mysqld_safe --skip-grant-tables &
```
之后，正常登陆数据库,
```
$ mysql -uroot 
```
登陆数据库之后，修改root用户密码，
```
mysql>use mysql;

mysql>update user set password=PASSWORD("mynewpassword") where User='root';

mysql>flush privileges;

mysql>exit;
```
最后，启动数据库，登陆即可.
```
$ sudo /etc/init.d/mysqld start

$ mysql -uroot -p
```

