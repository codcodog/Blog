---
title: php脚本自动化执行
date: 2016-07-11 21:16:33
tags: 
- 自动化
categories: phper
---
今天发现ｐｈｐ两个非常有意思的函数。通过这两个函数，可以实现类似ｓｈｅｌｌ脚本这样自动运行的功能，可以用来做商城后台的自动购买。　　

**ignore_user_abort — 设置客户端断开连接时是否中断脚本的执行**
```
int ignore_user_abort ([ string $value ] )  

设置客户端断开连接时是否中断脚本的执行

PHP以命令行脚本执行时，当脚本终端结束，脚本不会被立即中止，除非设置 value 为 TRUE，否则脚本输出任意字符时会被中止。
```
**set_time_limit — 设置脚本最大执行时间**
```
void set_time_limit ( int $seconds )

设置允许脚本运行的时间，单位为秒。如果超过了此设置，脚本返回一个致命的错误。默认值为30秒，或者是在php.ini的max_execution_time被定义的值，如果此值存在。

当此函数被调用时，set_time_limit()会从零开始重新启动超时计数器。换句话说，如果超时默认是30秒，在脚本运行了了25秒时调用 set_time_limit(20)，那么，脚本在超时之前可运行总时间为45秒。
```
把这两个函数结合到一起用，可以实现一些比较有趣的功能。
例如：
```
<?php
	set_time_limit(0);
	ignore_user_abort(TRUE);

	$i = 0;
	while(1){
		$i++;
		file_put_contents('1.txt',$i);
		
		sleep(3); //睡眠３秒钟
		if ($i > 20) break;
	}
?>
```
这个文件我命名为 `index.php` ，放在` www/`目录下，然后我在浏览器输入：`localhost/index.php` ，大概３秒钟，然后**关闭浏览器**。（这时候，ｐｈｐ脚本应该停止了，1.txt 文本内容应该是１）。

大概等待１分钟之后，去打开1.txt，发现里面内容是２１．　　
这就是这个函数结合使用的神奇之处了。 

**ignore_user_abort(true)**　让ｐｈｐ脚本在客户端断开了（关闭浏览器）依然继续运行。
**set_time_limit(0)**　让脚本执行时间没有限制（脚本一直运行）。

所以，如果需要用ｐｈｐ来写一些自动化功能程序的话，这两个函数可以满足脚本自动运行的功能需要。

