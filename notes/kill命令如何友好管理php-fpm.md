kill 命令如何友好管理 php-fpm
=============================

启动 `php-fpm`
```
$ /usr/local/php/sbin/php-fpm
```

重启 `php-fpm`
```
$ kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid`
```
> `php-fpm` 是一个支持 `USER2` 信号的进程管理器

关闭 `php-fpm`
```
$ kill -INT `cat /usr/local/php/var/run/php-fpm.pid`
```
或者
```
$ kill `cat /usr/local/php/var/run/php.pid`

$ cat /usr/local/php/var/run/php-fpm.pid | xargs kill
```
