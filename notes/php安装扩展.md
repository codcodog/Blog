---
title: php安装扩展
date: 2016-09-14 22:58:10
tags: 
- php extension
categories: phper
---

场景：编译安装了php，发现某个扩展没安装，如何添加扩展？  
> php无需重新编译安装，利用**phpize**可以像apache模块一样动态扩展.  

首先进入php安装的源码目录，例如：  
```
$ cd /usr/local/php   #这是php的安装目录
```
然后进入需要安装的扩展目录，例如，安装`pdo_mysql`扩展：  
```
$ cd ext/pdo_mysql
```
运行`phpize`:  
```
$ /usr/local/php/bin/phpize
```
编译安装：  
```
$ ./configure --with-php-config=/usr/local/php/bin/php-config
$ make
$ make install
```
这样，`pdo_mysql`扩展安装就完成了。  
编辑`php.ini`,去掉注释，开启相应的扩展即可。  
（PS: 需要注意的是安装扩展的目录路径是否跟`php.ini`默认的一致，如果不一致，则手动指定，例如：`extension_dir="/path/to/DIR"`）  

