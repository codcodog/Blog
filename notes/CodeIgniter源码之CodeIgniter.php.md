CodeIgniter源码之CodeIgniter.php
================================

`CodeIgniter.php`文件是CI框架的初始化文件，在框架启动的时候主要干了：  

- 声明框架版本

- 引入框架配置的常量：`constants.php`

- 引入框架公共函数库：`Common.php`

- 如果php版本低于`5.4`，则关闭魔术轉义

- 自定义错误函数处理

- `index.php`设置的`subclass_prefix`覆盖`config.php`文件的配置

- 是否使用`composer`第三方自动加载库`autoload`

- 加载`Benchmark`标记程序，计算框架性能

- 加载钩子，及其相关钩子函数：`Hooks`,`pre_system`

- 加载配置，并设置框架配置

- 设置字符集相关

- 加载兼容性功能：`mbstring`,`hash`

- 加载各种类/功能文件：`Utf8`,`URI`,`Router`,`Output`,`Security`,`Input`,`Lang`

- 调用钩子函数：`cache_override`，是否缓存显示

- 加载基础类：`Controller.php`，并单实例对象。

- 路由解释

- 执行一些钩子函数：`pre_controller`,`post_controller_constructor`,`post_controller`以及做一些时间标记

