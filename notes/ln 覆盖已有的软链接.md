ln 覆盖普通文件或目录
=====================

### 场景
`ln` 覆盖已存在的普通文件或目录.

### 创建软链接
创建一个普通文件的软链接
```
$ ln -s ~/.bashrc /tmp/test
$ ll /tmp/ | grep test
lrwxrwxrwx  1 cryven users        20 Jan 27 18:43 test -> /home/cryven/.bashrc
```

如果文件已存在，则会创建失败.
```
$ ln -s  ~/.vimrc /tmp/test
ln: failed to create symbolic link '/tmp/test': File exists
```

> 目录的软链接创建同上

### 覆盖已存在的文件

#### 普通文件软链接
使用 `-f` 选项覆盖已存在的普通文件.
```
$ ln -sf  ~/.vimrc /tmp/test
$ ll /tmp/ | grep test
lrwxrwxrwx  1 cryven users        19 Jan 27 18:52 test -> /home/cryven/.vimrc
```

#### 目录软链接
`-f` 选项在对目录的软链接覆盖，不会覆盖原来的，而是在此引用的目录里创建新的引用.
```
$ ln -s ~/demo/ /tmp/test1
$ ll /tmp/ | grep test1
lrwxrwxrwx  1 cryven users        18 Jan 27 18:56 test1 -> /home/cryven/demo/

$ ln -sf ~/.vim /tmp/test1
$ ll /tmp/ | grep test1
lrwxrwxrwx  1 cryven users        18 Jan 27 18:56 test1 -> /home/cryven/demo/

$ ll -a /tmp/test1/ | grep vim # .vim 的软链接创建在 test1 目录里了
lrwxrwxrwx  1 cryven users   18 Jan 27 19:05 .vim -> /home/cryven/.vim/
```

使用 `-n` 和 `-f` 选项来覆盖目录的软链接.
```
$ ln -snf ~/.vim /tmp/test1
$ ll /tmp/ | grep test1
lrwxrwxrwx 1 cryven users    17 Jan 27 19:11 test1 -> /home/cryven/.vim
```
> -n treat LINK_NAME as a normal file if it is a symbolic link to a directory
