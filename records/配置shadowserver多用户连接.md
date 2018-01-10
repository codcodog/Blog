---
title: shadowserver多用户连接
date: 2016-07-14 22:39:19
tags: 
- shadowserver
categories: Linux
---
在搬瓦工买了个ｖｐｓ，一键安装了ｓｈａｄｏｗｓｅｒｖｅｒ。之前一直是自己在用，没什么问题，速度也挺快的。最近朋友也想翻墙，然后我就在ｖｐｓ配置了下ｓｈａｄｏｗｓｅｒｖｅｒ的多用户连接，下面记录一下过程。

连上ｖｐｓ，然后：
```
# vim /etc/shadowsocks.json 
```
然后发现是一个新文件，没关系。在文件输入以下内容：
```
{
 "server":"my_server_ip"，
 "local_address": "127.0.0.1",
 "local_port":1080,
  "port_password": {
     "8381": "passwd1",
     "8382": "passwd2",
     "8383": "passwd3",
 },
 "timeout":300,
 "method":"aes-256-cfb",
 "fast_open": false
}
```
保存退出。
然后输入：
```
# ssserver -c /etc/shadowsocks.json
```
不过这是在前台运行的，后台运行的话，可以这样子：
```
开始：# ssserver -c /etc/shadowsocks.json -d start
结束：# ssserver -c /etc/shadowsocks.json -d stop
```
如果想开机启动的话，可以自己去搜索下相关帖子。因为博主的服务器，目前还关机过，就没继续折腾了。有兴趣的话，可以自己ｇｏｏｇｌｅ一下。

更多的问题，可以参考：[说明文档](https://github.com/shadowsocks/shadowsocks/wiki) 














