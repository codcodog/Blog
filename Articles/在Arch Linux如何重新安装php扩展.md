### 在 Arch Linux 下如何安装php扩展

场景
----

由于当初的`PHP`是`pacman`安装而不是源码安装的, 并且自己的Arch也好久没滚动更新了, 导致想安装PHP 扩展的时候:
```
$ sudo pacman -S php-sqlite
```
Error 404 找不到下载链接, 毕竟好久没更新系统了.  

而且又不是源码安装, 因此不能使用源码安装的办法, 详情请戳[PHP安装扩展](https://github.com/codcodog/Blog/issues/6)

如何安装扩展
------------

这里介绍的办法, 同上面的`PHP安装扩展`其实差不多, 只不过是多了几步而已.

首先查看自己的`PHP`版本,
```
$ php --version  
```
输出: 
```
PHP 7.0.9 (cli) (built: Jul 20 2016 17:12:28) ( NTS  )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
```

然后去`http://www.php.net/releases/`官网, 找到自己的`PHP`版本, 下载回来本地, 这里, 博主选择的是`7.0.9`版本.  

下载回来之后, 解压进入`ext`目录:
```
$ tar xfz php-7.0.9
$ cd php-7.0.9/ext/pod_sqlite/  # 博主安装pdo_sqlite这个扩展
```
然后在这个目录下运行: 
```
$ phpize
```
在这里可能会遇到类似这样的错误:
```
Cannot find config.m4. 
Make sure that you run '/opt/local/bin/phpize' in the top level source directory of the module
```

在该目录下, `ls` 可以看到一个`config0.m4`文件,
```
$ cp config0.m4 config.m4
```
然后依次运行: 
```
$ phpize
$ ./configure
$ make clean && make && make install 
```
然后我们需要的`pdo_sqlite.so`文件就生成在当前路径的`modules`目录下.

然后把`pdo_sqlite.so`复制到我们`pacman`安装的php动态库中, 在这里, 博主的是:
```
$ cp modules/pdo_sqlite.so /usr/lib/php/modules/
```

(PS: 如果不知道自己的动态库位置在那里, 在`php.ini`中打开`pdo_sqlite.so`扩展之后, 在命令行使用`php -m | grep sqlite`会收到相应的提示, 例如: `PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/lib/php/modules/pdo_sqlite.so'`)

最后, 重启下php服务, 尝试使用`pdo_sqlite.so`扩展, ok了!  
