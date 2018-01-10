---
title: zip解压中文乱码
date: 2016-11-18 16:54:34
tags: 
- Archlinux
categories: Linux
---
zip解压文件，中文乱码  
=====================
同事传了个zip文件给我，在Arch Linux解压，出现中文乱码，导致文件打开出错.  

Google了下，  
> 由于zip格式并没有指定编码格式，Windows下生成的zip文件中的编码是GBK/GB2312等，因此，导致zip文件在Linux下解压出现乱码问题，因为Linux下默认编码是UTF8.  

网上流传的一种解决办法是：  
> unzip -O cp936  

由于unzip没有`-O`这个选项，因此，在Arch Linux需要安装`unzip-iconv`.  

安装完后之后，使用`unzip -O cp936 FILE`就可以解压了.  
