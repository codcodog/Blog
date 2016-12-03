---
title: php基础常识
date: 2016-05-16 10:22:56
tags:
- php
- 基础
categories: phper
---
记录遇到的几个php基础常识。

##### 如何获取某个主页的内容，例如：获取 www.baidu.com 的内容 #####
答：这个问题本质上来说就是采集信息。有很多种办法，例如：cURL、file\_get\_contents、file、readfile等。在这里，就直接用file\_get\_contents()这个函数。

	<?php   
  	  $url = 'www.baidu.com';  //如果url中存在特殊字符，例如有空格，就需要使用urlencode()进行URL编码  
  	  $contents = file_get_contents($url);   
 	  echo $contetns;  
	?>    
##### error_reporting（）这个函数的作用 #####
答：error_reporting — 设置应该报告何种 PHP 错误。下面就列出一些常用的报错级别：   
>E\_ERROR：致命的运行时错误。这类错误一般是不可恢复的情况，例如内存分配导致的问题。后果是导致脚本终止不再继续运行。   
>E\_WARNING：运行时警告 (非致命错误)。仅给出提示信息，但是脚本不会终止运行。   
>E\_NOTICE： 运行时通知。表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知。   
>E\_ALL： E\_STRICT 出外的所有错误和警告信息。  
>E\_STRICT：启用 PHP 对代码的修改建议，以确保代码具有最佳的互操作性和向前兼容性。   
>E\_PARSE： 编译时语法解析错误。解析错误仅仅由分析器产生。   
>error_reporting ( 0 )：关闭所有PHP错误报告。  

##### echo、print、print\_r、printf、sprintf、var\_dump的区别 #####
>echo是php语言结构不是函数，输出一个或者多个字符串。  
>print也是语言结构不是函数 ，只允许输出一个字符串，返回值总为 1，速度上比echo慢。  
>print\_r：print\_r(mixed)打印关于变量的易于理解的信息。  
>var\_dump：显示关于一个或多个表达式的结构信息，包括表达式的类型与值。和print\_r()的区别，var\_dump返回表达式的类型与值而print\_r仅返回结果，相比调试代码使用var_dump更便于阅读。   
>printf：函数，把文字格式化以后输出。   
>sprintf：跟printf相似，但不打印，而是返回格式化后的文字，其他的与printf一样。 
  
##### php的常用魔术方法 #####
>\_\_construct（）：具有构造函数的类会在每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作。  
>\_\_destruct（）：析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。  
>\_\_call ( string $name , array $arguments )：在对象中调用一个不可访问方法时，\_\_call() 会被调用。   
>\_\_callStatic ( string $name , array $arguments )：用静态方式中调用一个不可访问方法时，\_\_callStatic() 会被调用。  
>\_\_get ( string $name )：读取不可访问属性的值时，\_\_get() 会被调用。   
>\_\_set ( string $name , mixed $value )：在给不可访问属性赋值时，\_\_set() 会被调用。  
>\_\_isset ( string $name )：当对不可访问属性调用 isset() 或 empty() 时，\_\_isset() 会被调用。  
>\_\_unset ( string $name )：当对不可访问属性调用 unset() 时，\_\_unset() 会被调用。  
>\_\_clone ( void )：当对象复制完成时，如果定义了 \_\_clone() 方法，则新创建的对象（复制生成的对象）中的 \_\_clone() 方法会被调用，可用于修改属性的值（如果有必要的话）。  

##### php中有那些魔术常量 #####
>\_\_LINE\_\_：文件中的当前行号。  
>\_\_FILE\_\_：文件的完整路径和文件名。  
>\_\_DIR\_\_：文件所在的目录。如果用在被包括文件中，则返回被包括的文件所在的目录。它等价于 dirname(\_\_FILE\_\_)。除非是根目录，否则目录中名不包括末尾的斜杠。  
>\_\_FUNCTION\_\_：函数名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该函数被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。  
>\_\_CLASS\_\_：类的名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该类被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。类名包括其被声明的作用区域（例如 Foo\Bar）。注意自 PHP 5.4 起 \_\_CLASS\_\_ 对 trait 也起作用。当用在 trait 方法中时，\_\_CLASS\_\_ 是调用 trait 方法的类的名字。  
>\_\_TRAIT\_\_： Trait 的名字（PHP 5.4.0 新加）。自 PHP 5.4 起此常量返回 trait 被定义时的名字（区分大小写）。Trait 名包括其被声明的作用区域（例如 Foo\Bar）  
>\_\_METHOD\_\_：类的方法名（PHP 5.0.0 新加）。返回该方法被定义时的名字（区分大小写）。  
>\_\_NAMESPACE\_\_：当前命名空间的名称（区分大小写）。此常量是在编译时定义的（PHP 5.3.0 新增）。
   
具体用法可以参考 [官方手册：php魔术常量](https://secure.php.net/manual/zh/language.constants.predefined.php "官方手册")  

##### php中抽象类跟接口的区别 #####
答：参考这篇博文：[PHP中的 抽象类（abstract class）和 接口（interface）](http://blog.csdn.net/sunlylorn/article/details/6124319 " PHP中的 抽象类（abstract class）和 接口（interface）")   

##### HTTP中GET、POST、HEAD的区别 #####
>GET： 请求指定的页面信息，并返回实体主体。  
>HEAD： 只请求页面的首部。  
>POST： 请求服务器接受所指定的文档作为对所标识的URI的新的从属实体。  
>PUT： 从客户端向服务器传送的数据取代指定的文档的内容。   
>DELETE： 请求服务器删除指定的页面。  

当服务器响应时，其状态行的信息为HTTP的版本号，状态码，及解释状态码的简单说明。现将5类状态码详细列出：  
① 客户方错误  
　　100　 继续  
　　101　 交换协议  
② 成功   
　　200 　OK  
　　201 　已创建  
　　202　 接收  
　　203　 非认证信息  
　　204　 无内容  
　　205 　重置内容  
　　206　 部分内容  
③ 重定向  
　　300 　多路选择  
　　301　 永久转移  
　　302　 暂时转移  
　　303　 参见其它  
　　304 　未修改（Not Modified）  
　　305　 使用**
④ 客户方错误  
　　400　 错误请求（Bad Request）  
　　401 　未认证  
　　402 　需要付费  
　　403　 禁止（Forbidden）  
　　404　 未找到（Not Found）  
　　405　 方法不允许  
　　406　 不接受  
　　407　 需要**认证  
　　408　 请求超时  
　　409　 冲突  
　　410 　失败  
　　411 　需要长度  
　　412　 条件失败  
　　413 　请求实体太大  
　　414 　请求URI太长  
　　415 　不支持媒体类型  
⑤ 服务器错误  
　　500　 服务器内部错误  
　　501　 未实现（Not Implemented）  
　　502　 网关失败  
　　504 　网关超时  
　　505 HTTP版本不支持  




