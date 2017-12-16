php -a 无法使用
================

问题：  
编译安装的 `PHP 7.1.12`， 在命令行输入 `php -a`, 提示 `Interactive mode enabled`, 无法使用.  

解决办法：  
安装 `readline` 扩展.
```
$ cd /usr/local/src/php-7.1.12/ext/readline
$ phpize
$ ./configure --with-php-config=/usr/local/php/bin/php-config
$ make && make install
$ make clean
```

编辑 `php.ini`， 添加 `extension=readline.so` 扩展.  

查看是否加载扩展
```
$ php -m | grep readline
readline
```

或者在编译安装的时候， 加上 `--with-readline` 参数.
