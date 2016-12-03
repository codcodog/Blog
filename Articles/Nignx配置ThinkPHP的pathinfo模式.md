---
title: Nignx配置ThinkPHP的pathinfo模式
date: 2016-05-11 17:09:39
tags: 
- Nginx
- ThinkPHP
categories:
- Nginx
---

> 把ThinkPHP的 URL_MODEL 设置为 2

    'URL_MODEL'  => '2', //URL模式
> Nginx rewrite 配置：

    location / { 
       if (!-e $request_filename) {
       rewrite  ^(.*)$  /index.php?s=$1  last;
       break;
      }
    }

> 如果你的ThinkPHP安装在二级目录，Nginx的伪静态方法设置如下，其中tp是所在的目录名称

    location /tp/ {
		if (!-e $request_filename){
		rewrite  ^/tp/(.*)$  /tp/index.php?s=$1  last;
	  }
    }