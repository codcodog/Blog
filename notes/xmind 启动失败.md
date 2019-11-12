xmind 启动失败
===============

#### 场景
Arch 系统安装了 `xmind`，启动失败，日志见 `~/.xmind/workspace/.metadata/.log` 文件.

#### 方案
Arch 系统自带的 `java` 版本太高导致 `xmind` 启动失败.

查看 `java` 版本
```
$ archlinux-java status
java-13-openjdk (default)
```

安装低版本 `java`
```
$ sudo pacman -S jdk8-openjdk
```

修改 `xmind` 配置文件
```
$ vi /usr/share/xmind/XMind/XMind.ini
```

删除 `--add-modules=java.se.ee` 选项

并且在配置文件最开头添加
```
-vm
/usr/lib/jvm/java-8-openjdk/bin
```

#### 参考
[xmind AUR](https://aur.archlinux.org/packages/xmind/)
