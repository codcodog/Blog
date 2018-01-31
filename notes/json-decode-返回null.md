---
title: json_decode()返回null
date: 2016-09-22 21:06:31
tags: 
- 规范
categories: Phper
---
在做接口的时候，利用php函数`json_decode()`解析传过来的json数据，却返回null.  
一开始还以为是接口没有数据过来，`var_dump()`了一下，结果有数据！ 但php解析却返回null, 很纳闷，于是Google了一下.  

`json_decode()`函数，要求输入的字符串是有要求的：  
> 使用UTF-8编码  
> 不能在最后元素有逗号  
> 不能使用单引号  
> 不能有\r,\t

所以，在接口传过来的json字符串中，需要查看是否包含了`BOM`头，如果有，去除.  

而博主遇到的并不是关于`BOM`头的问题，是编码问题，所以把接口传过来的`GBK`编码转成`UTF-8`编码就可以了. (PS: 在开发的时候还是要注重一些编程规范，例如，编码统一使用`UTF-8`)  

另外，可以使用`json_last_error()`, `json_last_error_msg()`等函数查看json解析错误的信息.  
