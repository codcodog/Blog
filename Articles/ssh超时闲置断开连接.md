ssh超时闲置断开连接
===================

当我们 `ssh` 某个服务器的时候， 闲置时间久了就会自动断开连接.  
为了避免断开连接，可以在客户端或者服务器端设置.

客户端设置
----------
在本地系统设置
```
$ vi /etc/ssh/ssh_config
```
添加如下内容
```
ServerAliveInterval 60
```
> 表示该系统`ssh`连接到某服务器的时候，每60s会发一个`KeepAlive`请求，防止断开连接.

服务器端设置
------------
在服务器端设置
```
$ vi /etc/ssh/ssh_config
```
添加如下内容
```
ClientAliveInterval 60
```
> 每一个连接到该服务器端的客户端都会受影响.

最后重启 `sshd` 服务即可
```
$ systemctl restart sshd
```
