foreach 空数组
==============

在研究`monolog`源码的时候, 发现了一个有趣的东东, 特此记录一下.  

在`monolog`的`Logger.php`文件大概332行左右, 发现了如下一段代码: 
```
foreach ($this->processors as $processor) {
    $record = call_user_func($processor, $record);
}
```

根据追踪的代码看来, `$this->processors`是一个空数组, 那么上面的这段代码有何作用呢?  
对此, 打了个断点.
```
echo 1 . PHP_EOL;
var_export($this->processors) . PHP_EOL;
foreach ($this->processors as $processor) {
    echo 2;
    $record = call_user_func($processor, $record);
    var_export($record);
}
echo 3 . PHP_EOL;die;
```

执行文件发现:  
```
PHP $ php -f demo.php 
1
array (

)3
PHP $ 
```
`foreach`代码块里的代码根本就没有执行.  

也就是说, 当`foreach`一个空数组的时候, 那么`foreach`的语句块的代码是不执行的.
查了下手册, 手册上没有说到这个奇怪的现象, 特别地记录一下.
