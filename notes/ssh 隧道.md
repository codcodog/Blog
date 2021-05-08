ssh 隧道
=========

`ssh` 隧道，或者叫 `ssh` 端口转发，主要有三种类型：
- 本地端口转发
- 远程端口转发
- 动态端口转发


### 本地端口转发

本地端口转发可以将本地（`ssh` 客户端）计算机上的端口请求转发到服务器端（`ssh` 服务端），  
再由服务器端转发到目标机器的端口上.

```
ssh -L [LOCAL_IP:]LOCAL_PORT:DESTINATION:DESTINATION_PORT [USER@]SSH_SERVER
```


### 远程端口转发

远程端口转发与本地端口转发相反，是将 `ssh` 服务器端侦听的端口请求转发到本地（`ssh` 客户端）计算机，  
再由本地计算机转发到目标机器的端口上.

```
ssh -R [REMOTE:]REMOTE_PORT:DESTINATION:DESTINATION_PORT [USER@]SSH_SERVER
```


### 动态端口转发

动态端口转发可以充当一个简单的 `socks` 代理服务器.  
当客户端连接到该端口时，连接将转发到服务器端（`ssh` 服务端），再由服务器端转发到目标计算机上的端口.

```
ssh -D [LOCAL_IP:]LOCAL_PORT [USER@]SSH_SERVER
```

例如，在端口 `1080` 创建 `socks` 隧道，再配置好 `SwitchyOmega`，就可以愉快地爬梯子了.
```
$ ssh -D 1080 -N -f user@server
```
> -N 不要执行远程命令  
> -f ssh 命令后台运行


### 应用场景

有 A, B, C 三台服务器，其中 A, C 网络不通，而 A, B 可以连接，B, C 网络也可以连接.

在 A 服务器上有个程序 a
```
$ curl localhost:8080/a
Hello World.
```

若要 C 服务器也能访问 A 服务上的程序 a，则需要通过 B 服务器进行中转.

在 A 服务器上进行远程端口转发
```
$ ssh -R 8080:127.0.0.1:8080 -N -f root@B
```
这个命令就是将 B 服务器上 8080 端口的请求转发到 A 服务器上，然后 A 再转发到 `127.0.0.1:8080`.

这样 C 通过 B 服务器就可以访问到 A 上的程序 a 了
```
$ curl http://B:8080/a
Hello World.
```

**注意**  
若端口转发失败，则 `ssh` 服务器端 `/etc/ssh/sshd_config` 需要配置 `GatewayPorts` 为 `yes`，再重启 `ssh`
```
$ systemctl restart sshd.service
```

### 参考
[如何设置SSH隧道](https://www.myfreax.com/how-to-setup-ssh-tunneling/)
