---
title: list函数的神奇用法
date: 2016-07-12 21:12:38
tags:
- php基础
categories: phper
---
发现　`ｌｉｓｔ（）`　函数的一个神奇用法。

一般，我们要交换两个数，是用下面这种方法的：
```
<?php
function swap(&$x,&$y){
	$tmp = $x;
	$x = $y;
	$y = $tmp;
}
	$a = 1;
	$b = 2;
	
	swap($a,$b);

	echo $a; //a=2
	echo $b; //b=1

?>
```
而，如果用 `list()`　函数来做的话，非常简单。
```
<?php

	$a =1;
  	$b = 2;
	list($a,$b) = array($b,$a);

	echo $a; //a=2
	echo $b; //b=1

?>
```
相对于 `swap()` 函数来说的话，`list()`函数语义清晰，代码简洁。但性能的话，还是`swap()`函数高。
