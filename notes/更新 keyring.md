更新 keyring
=============

### 场景
`Arch` 许久不更新，安装软件报 `error`.
```bash
$ sudo pacman -Sy docker # -y 参数是许久没更新系统，从服务器下载最新的包数据库到本地
...
...
error: containerd: signature from "Eli Schwartz <eschwartz@archlinux.org>" is unknown trust
:: File /var/cache/pacman/pkg/containerd-1.2.0-3-x86_64.pkg.tar.xz is corrupted (invalid or corrupted package (PGP signature)).
Do you want to delete it? [Y/n] y
error: failed to commit transaction (invalid or corrupted package)
Errors occurred, no packages were upgraded.
```

### 方案
更新 `keyring`
```bash
$ sudo pacman -S archlinux-keyring
```

然后再安装软件即可.
