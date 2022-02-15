systemd
========

`systemd` 是 Linux 控制系统和管理服务的工具，基本上所有的发行版都支持使用.

### 基础

#### 启动 / 停止

```bash
# 启动服务
$ systemctl start application.service

# 停止服务
$ systemctl stop application.service

# 重启服务
$ systemctl restart application.service

# 查看服务状态
$ systemctl status application.service

# 杀死服务的所有子进程
$ systemctl kill application.service
```

#### 重新加载配置
```bash
# 重新加载某服务配置文件
$ systemctl reload application.service

# 重新加载所有 unit 服务的配置文件
$ systemctl daemon-reload
```

##### `reload` 和 `daemon-reload` 的区别

`reload` 是重新加载某服务的配置，例如：apache 有 `httpd.conf` 配置，  `reload` 的时候，`systemd` 会发送一个 `SIGHUP` 信号让 `apache` 重新加载 `httpd.conf` 配置.   

`daemon-reload` 则是重新加载 `systemd` 的配置文件，例如：某服务 `application.service` 文件在 `/etc/systemd/systemd/` 目录下，设置5秒超时时间，则需要在 `application.service` 文件添加 `TimeoutSec`
```
$ systemctl cat application.service
[Unit]
Description=ATD daemon

[Service]
Type=forking
ExecStart=/usr/bin/atd
TimeoutSec=5

[Install]
WantedBy=multi-user.target
```
修改完之后，在服务不退出的情况下，使用 `daemon-reload` 重新加载下 `systemd` 服务文件配置.
> `/etc/systemd/systemd/` 目录下文件发生变更之后，都应该 `daemon-reload` 一下

#### 开机启动

```bash
# 开机启动
$ systemctl enable application.service

# 禁止开机启动
$ systemctl disable application.service
```

`enable` 通常是从 `/lib/systemd/system/` 或者 `/etc/systemd/system/`   
创建一个软链接到 `/etc/systemd/system/some_target.target.wants`
```
~ $ ll /etc/systemd/system/multi-user.target.wants/
total 0
lrwxrwxrwx 1 root root 40 Feb 13 18:25 openntpd.service -> /usr/lib/systemd/system/openntpd.service
lrwxrwxrwx 1 root root 40 Feb 13 18:19 remote-fs.target -> /usr/lib/systemd/system/remote-fs.target
```

#### 查看日志

```bash
# 查看系统所有服务日志
$ journalctl

# 查看某服务日志
$ journalctl -u application.service

# 实时滚动日志
$ journalctl -u application.service -f

# 查看尾部最新日志，不指定，默认10行
$ journalctl -u application.service -n 20
```

### 配置文件

配置文件主要在 `/usr/lib/systemd/system/` 或者 `/etc/systemd/system/`.

```
$ systemctl cat sshd.service

[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
> 配置文件的区块名和字段名，都是大小写敏感

#### [Unit] 块

- `Description` 简短描述
- `Documentation` 文档地址
- `Requires` 当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
- `Wants` 与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
- `BindsTo` 与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
- `Before` 如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
- `After` 如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
- `Conflicts` 这里指定的 Unit 不能与当前 Unit 同时运行
- ...

#### [Service] 块

`[Service]` 区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块.

- `Type` 定义了启动时进程的行为
    + simple 默认值，执行 ExecStart 指定的命令，启动主进程
    + forking 以 fork 方式从父进程创建子进程，创建后父进程会立即退出
    + oneshot 一次性进程，Systemd 会等当前服务退出，再继续往下执行
    + dbus 当前服务通过D-Bus启动
    + notify 当前服务启动完毕，会通知 Systemd，再继续往下执行
    + idle 若有其他任务执行完毕，当前服务才会运行
- `ExecStart` 启动当前服务的命令
- `ExecStartPre` 启动当前服务之前执行的命令
- `ExecStartPost` 启动当前服务之后执行的命令
- `ExecReload` 重启当前服务时执行的命令
- `ExecStop` 停止当前服务时执行的命令
- `ExecStopPost` 停止当前服务之后执行的命令
- `KillMode` Systemd 如何停止服务
    + control-group 默认值，当前控制组里面的所有子进程，都会被杀掉
    + process 只杀主进程
    + mixed 主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
    + none 没有进程会被杀掉，只是执行服务的 stop 命令
- `RestartSec` 自动重启当前服务间隔的秒数
- `Restart` 定义何种情况 Systemd 会自动重启当前服务
    + no 默认值，退出后不会重启
    + always 不管是什么退出原因，总是重启
    + on-success 只有正常退出时（退出状态码为0），才会重启
    + on-failure 非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启
    + on-abnormal 只有被信号终止和超时，才会重启
    + on-abort 只有在收到没有捕捉到的信号终止时，才会重启
    + on-watchdog 超时退出，才会重启
- `TimeoutSec` 定义 Systemd 停止当前服务之前等待的秒数
- `Environment` 指定环境变量


#### [Install] 块

定义如何启动，以及是否开机启动.

- `WantedBy` 它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入 `/etc/systemd/system` 目录下面以 Target 名 + .wants后缀构成的子目录中
- `RequiredBy` 它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入 `/etc/systemd/system` 目录下面以 Target 名 + .required后缀构成的子目录中
- `Alias` 当前 Unit 可用于启动的别名
- `Also` 当前 Unit 激活（enable）时，会被同时激活的其他 Unit

### Target

启动计算机的时候，需要启动大量的 Unit。如果每一次启动，都要一一写明本次启动需要哪些 Unit，显然非常不方便。Systemd 的解决方案就是 `Target`.

简单说，`Target` 就是一个 Unit 组，包含许多相关的 Unit。 启动某个 `Target` 的时候，Systemd 就会启动里面所有的 Unit.

### 参考

- man systemctl
- man 5 systemd.unit
- [How To Use Systemctl to Manage Systemd Services and Units](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)
- [Systemd 入门教程：命令篇](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
