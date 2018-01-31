---
title: Arch Linux安装过程
date: 2016-09-08 11:13:32
tags: 
- Arch Linux
categories: Linux
---
最近倒腾了Arch Linux。记录一下安装过程。  

#### dd镜像 ####
第一步，做启动盘，这里用 `dd` 命令来做。
```
$ sudo umount /dev/sdX
$ sudo dd if=/path/to/archlinux.iso of=/dev/sdX bs=4M && sync
```
说明：
> sdX就是你的u盘，可以用`lsblk`查看
> The `sync` bit is important as `dd` can return before the write operation finishes.
> sync函数只是将所有修改过的块缓冲区排入写队列，然后就返回，它并不等待实际写磁盘操作结束。
> 通常称为update的系统守护进程会周期性地（一般每隔30秒）调用sync函数。这就保证了定期冲洗内核的块缓冲区。  

(PS: 有人说`sync`是为了确定u盘实际刻录完，也有人说加这个参数是没有什么作用)

#### 网络连接 ####
启动盘做好之后，就进入Arch live 版进行系统的安装。  
因为Arch Linux是在线安装的，所以需要先连上网络。  
由于，博主是使用wifi的，所以用了一下方式：  
```
# iwconfig #查看网线网卡
# wifi-menu wlp3s0b1 #wlp3s0b1是博主的无线网卡
```
输入密码，连上wifi，测试下网络：
```
# ping -c 3 www.archlinux.org
```

#### 分区 ####
连上网络，开始正式安装系统。
查看自己需要安装的系统盘是那个.
```
# lsblk
```
这里，`sda`为本地硬盘，然后进行分区。
```
# cfdis  /dev/sda
```
分好盘之后，格式化.
```
# mkfs.ext4 /dev/sda1
# mkfs.ext4 /dev/sda5
# mkswap /dev/sda6
```
挂载目录.
```
# mount /dev/sda1 /mnt
# mkdir /mnt/home
# mount /dev/sda5 /mnt/home
# swapon /dev/sda6
```

#### 配置镜像源 ####
用vi/nano打开镜像列表
```
# vi /etc/pacman.d/mirrorlist
```
添加国内镜像服务
```
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = http://mirrors.sohu.com/archlinux/$repo/os/$arch
```

#### 安装系统基础包 ####
```
# pacstrap -i /mnt base base-devel
```

#### 配置fstab ####
```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```
检测一下fstab内容，可以看到跟目录/和/home已经被挂载。

切换到新系统
```
# arch-chroot /mnt
```

#### 配置区域和语言 ####
配置系统字符集.(把en_US.UTF-8,zh_UTF-8前的#删除)
```
# vi /etc/locale.gen
```
生成locale文件
```
# locale-gen
# echo LANG=en_US.UTF-8 > /etc/locale.conf 
# export LANG=en_US.UTF-8
```

#### 设置时间 ####
查看提供的区域时间
```
# ls /usr/share/zoneinfo/
```
写入系统
```
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# hwclock --systohc --utc
```

#### 设置主机名和创建用户 ####
设置主机名
```
# echo codcodog > /etc/hostname
```
创建密码
```
# passwd
```
创建用户
```
# useradd -m -g users -G wheel,storage,power -s /bin/bash even
# passwd even
```
为用户设置超级权限
```
# pacman -S sudo
# EDITOR=nano visudo
```
去掉前面的注释
```
%whell ALL=(ALL) ALL
```

#### 安装系统开机需要使用的软件 ####
自动补全
```
# pacman -S bash-completion
```
网络连接
```
# pacman -S iw wpa_supplicant dialog
```
yaourt包管理.
```
# vi /etc/pacman.conf
```
加入以下内容
```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```
补充说明，在使用yaourt时，可能无法构建包或无法获取密钥环的情况，可以参考一下方法
```
# pacman -S packagekit
# pacmam -S archlinuxcn-keyring
# yaourt -Sy base-devel
```

#### 完成安装内容 ####
安装grub
```
# pacman -S grub
```
grub 启动引导至硬盘
```
# grub-install --target=i386-pc --recheck /dev/sda
```
识别多系统（可选安装）
```
# pacman -S os-prober
```
配置grub
```
# grub-mkconfig -o /boot/grub/grub.cfg
```
卸载重启
```
# exit
# umount -R /mnt
# reboot
```
至此，系统已经安装完成。  

#### 安装驱动，触摸板 ####
```
# pacman -S xorg-server xorg-server-utils xorg-xinit
```

安装显卡驱动，可以看Arch Linux WIKI.  

安装触摸板.
```
# pacman -S xf86-input-synaptics
```


最后备份一下自己常用的软件：
> awesome
> zsh
> oh-my-zsh
> conky
> fcitx
> ss-qt5
> chromium
> okular
> vim
> httpie
> ranger
> xmind
> xflux
> hexchat
> liboffice
> light(调节亮度)
> mysql-workbench
> Nginx+Php+MariaDB
> Hexo
> Git
> Svn
> Composer
> ctags

