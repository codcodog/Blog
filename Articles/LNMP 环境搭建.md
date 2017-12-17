LNMP 环境搭建和配置
===================

Nginx 安装
-----------

#### 下载源码
在 [Nginx](http://nginx.org/en/download.html) 官网，下载好源码包.
```
$ wget -c wget -c http://nginx.org/download/nginx-1.12.2.tar.gz
$ tar xvzf nginx-1.12.2.tar.gz
```

#### 安装依赖
在编译安装开始前， 先把依赖包安装上.
> ssl 功能需要 `openssl` 库  
> gzip 模块需要 `zlib` 库  
> rewrite 模块需要 `pcre` 库  

`Arch` 系统.
```
$ yaourt -S pcre zlib openssl
```

`Centos` 系统. (仅供参考)
```
$ yum install pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

#### 创建用户和用户组
```
$ groupadd nginx
$ which nologin # 查看 nologin 位置
/usr/bin/nologin
$ useradd -s /usr/bin/nologin -g nginx -M nginx
```

#### 开始安装
查看编译参数
```
$ cd nginx-1.12.2
$ ./configure --help
```

编译安装
```
$ ./configure --prefix=/usr/local/nginx \
> --user=nginx \
> --group=nginx \
> --with-openssl=/usr/bin/openssl
$ make && make install
$ make clean
```
> 这里使用 `--with-openssl` 是为了使用上 `openssl` 库.  
> 使用 `which openssl` 查看库位置.  
> 而 `zlib` 和 `pcre` 库默认启用，编译信息就可以看到.

```
Configuration summary
  + using system PCRE library
  + using OpenSSL library: /usr/bin/openssl
  + using system zlib library
```

最后，将 `Nginx` 主程序，添加一个软链接到 `$PATH` 其中一个路径目录.
```
$ echo $PATH
/usr/bin:/usr/local/sbin:/usr/local/bin
$ ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
```
> `ln -s` 最好使用绝对路径而不是相对路径，否则会出现一些莫名其妙的错误.

#### 配置
`nginx` 基本配置
```
# 定义用户和用户组
user nginx nginx;

# nginx 进程数，建议设置为CPU总核心数
worker_processes  4;

# 全局错误日志定义类型：debug, info, notice, warn, error, crit
error_log   logs/error_log  error;

# 进程文件
pid        logs/nginx.pid;

# 在http块输入，意思是搜索`conf.d`目录下的`.conf`文件同`nginx.conf`一起使用，方便模块化配置站点.
# 在 `usr/local/nginx` 需要创建 `conf.d` 目录.
include /usr/local/nginx/conf.d/*.conf;

# 在http块输入, 配置隐藏nginx版本号
server_tokens off;
```

配置支持 `PHP` 脚本
```
$ vi conf.d/localhost.conf
```
输入如下内容
```
server {
    listen  80;
    server_name localhost;

    location / {
        root /usr/local/nginx/html;
        index index.php index.html index.htm;

        # 隐藏index.php
        if (-f $request_filename/index.php){
            rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename){
            rewrite (.*) /index.php;
        }
    }

    location ~ \.php$ {
        root /usr/local/nginx/html;
        fastcgi_pass   unix:/usr/local/php/var/run/php-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_read_timeout 900;
        include        fastcgi_params;
    }
}
```

不过，在这之前， 需要在 `nginx.conf` 把原来的 `Server` 配置修改下，避免冲突.
```
server {
    listen       8080;
    server_name  localhost;
```

参考文章：[nginx编译安装](http://blog.51cto.com/meiling/1978442)

PHP 安装
--------

#### 下载源码
在 [PHP](http://php.net/downloads.php) 官网， 下载好源码包.
```
$ wget -c http://cn2.php.net/distributions/php-7.1.12.tar.gz
$ tar xvzf php-7.1.12.tar.gz 
```

#### 创建用户和用户组
```
$ useradd -s /usr/bin/nologin -M -U php-fpm
$ id php-fpm # 查看用户信息
uid=1002(php-fpm) gid=1002(php-fpm) groups=1002(php-fpm)
```

#### 编译安装
编译
```
$ ./configure --prefix=/usr/local/php \
> --enable-fpm \
> --with-fpm-user=php-fpm \
> --with-fpm-group=php-fpm \
> --with-config-file-path=/usr/local/php/etc \
> --with-config-file-scan-dir=/usr/local/php/etc/conf.d \
> --with-openssl \
> --with-curl \
> --with-gd \
> --with-jpeg-dir \
> --with-png-dir \
> --with-zlib \
> --with-freetype-dir \
> --enable-mbstring \
> --with-mcrypt \
> --with-mysqli \
> --enable-opcache \
> --with-pdo-mysql \
> --with-openssl \
> --enable-mysqlnd 
```

按照上面参数进行编译， 提示缺少依赖.
```
configure: error: mcrypt.h not found. Please reinstall libmcrypt.
```

安装依赖
```
$ yaourt -S libmcrypt
```

重新编译安装
```
$ ./configure --prefix=/usr/local/php \
... (参数同上)
$ make && make install
$ make clean
```

安装完成后， 把 `PHP` 命令路径加进 `PATH`.
```
$ vi .bashrc
```

输入
```
export PATH="$PATH:/usr/local/php/bin:/usr/local/php/sbin"
```

重新加载文件
```
$ source .bashrc
$ php --version
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies
```

#### 配置
`php.ini` 文件
```
$ cp php.ini-development /usr/local/php/etc/php.ini
```

创建 `conf.d` 目录
```
$ mkdir /usr/local/php/etc/conf.d
```
> 这个目录对应 `--with-config-file-scan-dir` 编译参数  
> 作用是搜索该目录下的 `ini` 文件 和 `php.ini` 一起使用  
> 方便模块化配置那些扩展 `extension=xx.so`, 或者利用脚本自动化生成配置文件.

`php-fpm.conf` 文件
```
$ cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```

`www.conf` 文件
```
$ cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
```

编辑 `php.ini`
```
# 关闭PHP版本信息
expose_php = off
```

编辑 `php-fpm.conf`
```
# pid 设置
pid = run/php-fpm.pid  

# 错误日志, 日志等级
# 等级级别：alert(须立即处理), error(错误), warning(警告), notice(一般重要信息), debug(调试信息)
error_log = log/php-fpm.log
log_level = error
```

编辑 `www.conf`
> `www.conf` 其实属于 `php-fpm.conf`  
> 因为在 `php-fpm.conf` 最后一行: `include=/usr/local/php/etc/php-fpm.d/*.conf`

```
# 启动进程的用户和用户组
user = php-fpm
group = php-fpm

# 当 php 和 nginx 同一个服务器的时候，设置为 nginx 的 socket 文件
;listen = 127.0.0.1:9000 
listen = /usr/local/php/var/run/php-fpm.sock

# 设置 unix socket 的用户, 用户组，权限
listen.owner = nginx
listen.group = nginx
listen.mode = 0666
```

关于 `PHP` 进程配置优化
```
pm = dynamic # php-fpm 进程启动模式
pm.max_children = 50 # 子进程最大数
pm.start_servers = 2 # 启动时的进程数，默认值：min_spare_servers + (max_spare_servers - min_spare_servers) / 2
pm.min_spare_servers = 1 # 保证空闲进程数最小值
pm.max_spare_servers = 3 # 保证空闲进程数最大值
pm.max_requests = 500 # 设置每个子进程重生之前服务的请求数. 对于可能存在内存泄露的第三方模块非常有用.
pm.status_path = /status
```
`pm` 可以设置成: `static`, `dynamic`, `ondemand`  

`pm = static` 表示创建的 `php-fpm` 子进程数是固定的，只有 `pm.max_children = 50` 这个参数生效，也就是说，在启动 `php-fpm` 的时候，会一起启动51个(主 + 50子)进程.

`pm = dynamic` 表示启动进程是动态分配的， 随着请求量动态变化的.  
由 `pm.max_children`, `pm.start_servers`, `pm.min_spare_servers`, `pm.max_spare_servers` 这几个参数共同决定.

> 一般规则： 动态适合小内存机器，灵活分配进程， 省内存。静态适合大内存机器.

参考文章：  
[PHP-FPM With PHP7 From Source](https://linuxadmin.io/php-fpm-php7-source/)  
[Linux 下编译安装 PHP 5.6](http://blog.aboutc.net/linux/65/compile-and-install-php-on-linux)  
[php-fpm的配置和优化](https://shaohualee.com/article/758)
