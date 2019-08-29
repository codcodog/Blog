mysql-workbench not work
========================

### 场景
Arch 系统 `mysql-workbench` 连接数据库时报 
```
The name org.freedesktop.secrets was not provided by any .service files.
org.freedesktop.secrets: No such file or directory
```

### 方案
这是 `mysql-workbench` 的一个 `bug`，需要安装 `gnome-keyring`
```
$ yay -S gnome-keyring
```
> 详情查看 https://bugs.mysql.com/bug.php?id=92183
