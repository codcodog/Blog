---
title: TP开发关于项目中目录相对位置问题
date: 2016-07-10 14:45:33
tags: 
- TP开发
- 基础
categories: phper
---

因为项目中某个功能，需要修改项目中公共模块里的配置文件config.php，它的位置在
*www/wenda(项目名称)/Application/Common/Conf/config.php*  
而我的功能文件在  
*www/wenda/Application/Admin/Controller/RewardController.class.php*  

在TP中，整个目录结构（在项目wenda文件夹下）是这样的：
Application
——Admin
——Home
——Common
Public
ThinkPHP
Uploads
index.php

其中，Admin，Home，Common是在Application目录下的。我在*RewardController.class.php*是这样引用*config.php*的：
```
$file = require('../../Common/Conf/config.php');
```
结果出现：
>require(): Failed opening required '../../Common/Conf/config.php' (include_path='.:/php/includes')
>错误位置
>FILE: /var/www/html/wenda/Application/Admin/Controller/RewardController.class.php 　LINE: 16

我的相对目录层级是完全没问题的，但是config.php文件就是引不进来，那肯定是引进的路径不正确了。　　
于是，Ｇｏｏｇｌｅ了一下，结果还真是......
>在Ｔｐ中，**所有的路径都是相对index.php的**

于是，我修改一下代码，如下：
```
$file = require('./Application/Common/Conf/config.php');
```
*config.php*文件能正常被require了！！！














