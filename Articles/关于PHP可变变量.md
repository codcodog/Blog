关于PHP可变变量
===============

在开发一个Api的时候, 同样的代码, 在本地 (PHP 7.0) 和服务器 (PHP 5.4) 运行, 结果却不一样.  
代码如下(截取关键部分): 
```PHP
if ( ! empty($flag) && $flag != $$key ) {
    $end_index = count($$key) - 1;
    $$key[$end_index] = end($$flag);
}
```

一步一步断点排查之后, 发现问题出在 `$$key[$end_index]` 这里.  
查询了一下手册, 发现在使用`可变变量`的时候, 需要注意语义的区分, 特别是在对数组使用的时候.  

在这里 `$$key[$end_index]` 有两种意思: `${$key}[$end_index]` 和 `${$key[$end_index]}`.  

而我遇到的问题就是, 本地和服务器在解释 `$$key[$end_index]` 就是刚好这两种情况, 导致本地和服务器运行的结果出现差异.

最后, 修正之后的代码如下:  
```PHP
if ( ! empty($flag) && $flag != $$key ) {
    $end_index = count($$key) - 1;
    ${$key}[$end_index] = end($$flag);
}
```


具体可以参考PHP手册: [可变变量](http://php.net/manual/zh/language.variables.variable.php)

