---
title: '初识dpkg,apt-get,aptitude'
date: 2016-11-23 15:40:09
tags: 
- Debian
- 包管理工具
categories: Linux
---

区别
====

`dpkg`绕过apt包管理数据库对软件包进行操作，所以用`dpkg`安装的软件可以用apt再安装一次，系统不知道之前已经安装过了，将会覆盖之前`dpkg`的安装.  
`dpkg`是用来安装.deb文件，但不会解决包依赖问题，且不关心软件仓库的软件，可以用于安装本地的deb文件，类似`rpm`.  

`apt`会解决和安装模块的依赖问题，并会咨询软件仓库，但不会安装本地的deb文件，`apt`是建立在dpkg之上的软件管理工具.  

`aptitude`与apt-get一样，是Debian及其衍生系统极其强大的包管理工具. 与apt-get不同的是，`aptitude`在处理以来问题上更佳一些. 例如，`aptitude`在删除一个包时，会同时删除本身所依赖的包.  

常用命令
========

dpkg
----

> dpkg -l 查询deb包详细信息，没有指定包则显示全部已安装的包  
>
> dpkg -s PACKAGE 查看已经安装的指定软件包的详细信息  
>
> dpkg -L PACKAGE 列出一个包安装的所有文件清单  
>
> dpkg -S PACKAGE 查看系统中的某个文件属于那个软件包  
>
> dpkg -i PACKAGE deb包安装  
>
> dpkg -r PACKAGE deb包卸载  
>
> dpkg -P PACKAGE 彻底卸载，包括软件的配置文件  
>
> dpkg -c PACKAGE 查询deb包包含的文件  
>
> dpkg -L 查看系统中安装的详细清单  

apt-get
-------

> apt-get update 在修改`/etc/apt/sources.list`或者`/etc/apt/preferences`之后运行该命令。此外您需要定期运行这一命令以确保您的软件包列表是最新的  
>
> apt-get install PACKAGE 安装一个新软件包  
>
> apt-get remove PACKAGE 卸载一个已安装的包（保留配置文件）  
>
> apt-get -purge remove PACKAGE 卸载一个已安装的包（删除配置文件）  
>
> apt-get upgrade 更新所有已安装的软件包  
>
> apt-get dist-upgrade 将系统升级到最新版本  
>
> apt-cache search KEYWORDS 用关键字搜索包  
>
> apt-cache show PACKAGE 显示特定包的信息  
>
> apt-cache depends PACKAGE 显示包的依赖  

aptitude
--------

> aptitude update 更新可用的包列表  
>
> aptitude upgrade 升级可用的包  
>
> aptitude dist-upgrade 将系统升级到最新的发行版  
>
> aptitude install PACKAGE 安装包  
>
> aptitude remove PACKAGE 删除包  
>
> aptitude purge PACKAGE 删除包及其配置文件  
>
> aptitude search STRING 搜索包  
>
> aptitude show PACKAGE 显示包的详细信息  
>
> aptitude clean 删除下载的包文件  
>
> aptitude autoclean 仅删除过期的包文件  
