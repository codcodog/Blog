---
title: Nginx配置多站点需要踩的坑
date: 2016-08-02 23:40:07
tags: 
- Nginx
categories: Linux
---
从Windows下的Apache转移到Linux下的Nginx，发现有很多坑需要踩。  
以下就做个简单的记录，方便后来者爬坑。  

配置Nginx，一般会遇到以下几个坑：  
    - 配置nginx支持pathinfo模式  
    - 配置友好URL，隐藏index.php  
    - 同一服务器配置多站点  

#### 配置pathinfo模式 #### 
在server（也就是你的站点，一个server对应一个站点）中输入以下内容：  
```
location ~ ^(.+\.php)(.*)$ {
    root html/[站点目录]; #配置站点目录路径
    fastcgi_split_path_info ^(.+\.php)(.*)$;
    fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_script_name;
}
```
其中``root html/[站点目录]``这个一定要填写跟你server的站点目录路径，要不它默认为``html``，从而导致路径解析失败。会出现的现象就是，在之前配置的``location``中定义了`root`路径也无法生效,访问`http://localhost`会跳转到`/usr/local/nginx/html/index.php[index.html]`,因为我的nginx是源码安装，所以路径可能不同，yum安装的话一般会在`/usr/share/nginx/html/index.php[index.html]`.  
(PS：原默认关于fastcgi的配置可以注释掉)
#### 隐藏index.php ####
隐藏index.php，大多数是采用Nginx的重写规则来进行的。  
下面，就是博主的列出的一个参考：  
```
location / {
    root    html/[站点目录];
    index   index.php;
    
    if (!-e $request_filename) {
        rewrite ^(.*)$ /index.php/$1;
    }
}
```
这里的站点目录是你程序（框架）的index.php所在的目录。例如，CI框架的话，`root html/ci`，其中index.php位于`html/ci/index.php`.  
本质上，隐藏index.php文件就是重写URL。具体详细用法可以参考Nginx重写模块的官方文档：[Module ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite)  

#### 同一服务器多站点配置 ####
一般一台服务器不会单一的运行一个站点，往往是运行多个站点的。  
在Nginx配置多站点是非常简单，便捷的。正如，前面所说的，一个server对应一个站点。例如：
```
server {
    listen  80;
    server_name SERVER_NAME1;
    location / {
    ....
    }
}
server {
    listen  80;
    server_name SERVER_NAME2;
    location / {
    ....
    }
}
```
这样，就配置了两个站点，分别为`SERVER_NAME1`和`SERVER_NAME2`.  

最后，附上博主的一份配置文件以供参考：
```
user  nginx nginx;
worker_processes  2;

#error_log  logs/error.log;
error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


 http {
    include       mime.types;
    default_type  application/octet-stream;

#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
#                  '$status $body_bytes_sent "$http_referer" '
#                  '"$http_user_agent" "$http_x_forwarded_for"';

#access_log  logs/access.log  main;
    rewrite_log on;

    sendfile        on;
#tcp_nopush     on;

#keepalive_timeout  0;
    keepalive_timeout  65;

#gzip  on;
    server {
        listen  80;
        server_name     pay.zf2.com;

        location / {
            root    html/zf2/pay/public;
            index   index.php index.html index.htm; 

            if (!-e $request_filename){
                rewrite ^/(.*)$ /index.php/$1;
            }
        }

        location ~ ^(.+\.php)(.*)$ {
            root html/zf2/pay/public;
            fastcgi_split_path_info ^(.+\.php)(.*)$;
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock; 
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_script_name;
        }
    }

    server {
        listen 80;
        server_name mp.zf2.com;

        location / {
            root    html/zf2/server/public;
            index   index.php index.html index.htm;

            if (!-e $request_filename){
            rewrite ^(.*)$ /index.php/$1;
            }
        }

        location ~ ^(.+\.php)(.*)$ {
            root html/zf2/server/public;
            fastcgi_split_path_info ^(.+\.php)(.*)$;
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock; 
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_script_name;
        }
    }
}
```

