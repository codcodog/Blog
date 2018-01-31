---
title: '了解php://input'
date: 2016-07-05 20:48:14
tags: 
- php基础
categories: phper
---
之前就看到过 ** php://input **，不怎么理解，今天特意去查了一下，遂做下笔记。  

首先，先了解一下 <form\>标签的enctype属性。  
>**application/x-www-form-urlencoded** : 默认模式，在发送到服务器之前，所有字符都会进行编码（空格转换为 “+” 加号，特殊符号转换为 ASCII HEX 值）  
**multipart/form-data** : 不对字符编码，在使用包含文件上传控件的表单时，必须使用该值。  
**text/plain** : 空格转换为 “+” 加号，但不对特殊字符编码。  

当在网页提交了一个表单之后，可以使用三种 PHP 方式来获取 Post 数据：``$_POST``，``$HTTP_RAW_POST_DATA ``和 ``php://input``，有什么区别呢？

- $_POST 
``$_POST ``是获取表单 POST 过来数据（body部分）的最常用方法，上传的文件信息使用 $_FILES 获取。

- $HTTP_RAW_POST_DATA  
当浏览器从表单发送 POST 请求的时候，默认的 media type 是 “application/x-www-form-urlencoded”，意思就是字段名和值都编码了，每个 key-value 对使用 ‘&’ 字符分隔开，key 和 value 使用 ‘=’ 分开，并且 key 和 value 中的空格都会被替换成 + ，其他特殊字符都会被使用 urlencode 方式进行编码。  

比如下面的 key-value 对：
>name: Jonathan Doe
>age: 23
>formula: a + b == 13%!

会被编码下面的原始数据：  
>name=Jonathan+Doe&age=23&formula=a+%2B+b+%3D%3D+13%25%21

PHP 会解析这些原始的 POST 数据，并且格式化成数组，填充到 $_POST 中： 

	Array
	(
	    [name] => Jonathan Doe
	    [age] => 23
	    [formula] => a + b == 13%!
	)

`$HTTP_RAW_POST_DATA` 是 PHP 的一个预定义的变量，用来获取原始的 POST 数据，比如上面的情况下，$HTTP_RAW_POST_DATA 的值就是：
>name=Jonathan+Doe&age=23&formula=a+%2B+b+%3D%3D+13%25%21

但是 $HTTP_RAW_POST_DATA 需要在 php.ini 中设置开启：
**always_populate_raw_post_data = On	**  
还有一点，`$HTTP_RAW_POST_DATA `不支持 enctype=”multipart/form-data” 方式传递的数据，这种情况下，我们要用 `$_POST` 获取字段的内容，$_FILES 来获取上传的文件信息。

- php://input

由于 $HTTP_RAW_POST_DATA 取决于 php.ini 设置，有没有更好的方法呢？

我们可以使用 php://input 来获取原始的 POST 数据，并且 php://input 比`$HTTP_RAW_POST_DATA `更少消耗内存，当然 php://input 和 $HTTP_RAW_POST_DATA 一样，它也不支持 enctype=”multipart/form-data” 方式传递的数据。

由于 php://input 只是数据流，我们可以使用 file_get_contents() 函数去获取它的内容：

	$post_data = file_get_contents('php://input');
	print_r($post_data);

获取到的内容和 $HTTP_RAW_POST_DATA 是一样的。   

**那么原始的 POST 数据有什么用？因为很多时候，接收到不是网页 POST 过来的数据，而是可能通过其他方式 POST 过来的 “text/xml” 格式的数据，这些内容无法解析成 $_POST 数组，这个时候我们就需要原始的 POST 数据进行处理。**

