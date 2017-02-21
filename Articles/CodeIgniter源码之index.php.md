CodeIgniter源码之index.php
==========================

`index.php`基本是每个框架的入口文件，而`CodeIgniter`框架的`index.php`文件主要是设置和定义了一些常量，如下：  

1. 定义开发环境常量：  

- ENVIRONMENT：开发环境

```
define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'development');
```

> `$_SERVER['CI_ENV']`是在Apache服务器配置的变量。

另外，对于`error_reporting`函数的用法不太理解：  

```
error_reporting(E_ALL & ~E_NOTICE & ~E_DEPRECATED & ~E_STRICT & ~E_USER_NOTICE & ~E_USER_DEPRECATED);
```

Google了下，参考这篇文章：[你真的了解error_reporting错误级别的设置么](https://www.iwantmoney.cn/article/view?id=22)  
才对`error_reporting`函数的用法有个清晰的认知。  

2. 定义常用目录路径常量：  

- SELF：自身文件名：`index.php`

- BASEPATH：`system`目录路径：`/usr/local/nginx/html/CodeIgniter-3.1.2/system/`

- FCPATH：前台控制器（此文件）目录的路径，即index文件所在的目录路径：`/usr/local/nginx/html/CodeIgniter-3.1.2/`

- SYSDIR：`system`目录名：`system`

- APPPATH：`application`目录路径：`/usr/local/nginx/html/CodeIgniter-3.1.2/application/`

- VIEWPATH：`view`目录路径：`/usr/local/nginx/html/CodeIgniter-3.1.2/application/views/`

最后引入基础类`CodeIgniter`文件：`require_once BASEPATH.'core/CodeIgniter.php'`  

