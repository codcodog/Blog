Arch 编译安装 PHP 失败
=======================

### 场景
在新安装的 `Arch` 系统编译安装 `PHP` 失败，提示错误
```
configure: error: freetype-config not found.
```

编译安装的参数如下
```bash
sudo  ./configure --prefix=/usr/local/php 
--enable-fpm 
--with-fpm-user=php-fpm 
--with-fpm-group=php-fpm 
--with-config-file-path=/usr/local/php/etc 
--with-config-file-scan-dir=/usr/local/php/etc/conf.d 
--with-openssl 
--with-curl 
--with-gd 
--with-jpeg-dir 
--with-png-dir 
--with-zlib 
--with-freetype-dir=/usr 
--enable-mbstring 
--with-mysqli 
--enable-opcache 
--with-pdo-mysql 
--with-openssl 
--enable-mysqlnd  
--with-webp-dir 
--with-xpm-dir
```

### 原因
查找资料发现
```
They should really stop using this gross *-config script and start using pkg-config properly.
```
`freetype-config` 不再使用，而是使用 `pkg-config`  
由于 `php/ext/gd/config.m4` 还是使用 `freetype-config` 而不是 `pkg-config`  
导致报 `configure: error: freetype-config not found.` 错误.
> 详细查看 php/ext/gd/config.m4 脚本


#### 解决办法
创建一个文件 `/usr/sbin/freetype-config`
```bash
#!/bin/sh

/usr/sbin/pkg-config freetype2 $@
```

重新编译安装，就可以了.

*参考资料*  
[things fail to build because freetype-config no longer exists](https://bugs.archlinux.org/task/58447)  
[AUR(en)-php71](https://aur.archlinux.org/packages/php71/)
