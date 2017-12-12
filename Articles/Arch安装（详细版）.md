Arch 安装和个人配置记录
=======================

### 制作启动盘
类`Unix`系统， 使用`dd`命令制作U盘启动
```
$ dd if=/path/to/archlinux.iso of=/dev/sdX bs=4M && sync
```

Wins系统， 则可以用 [Rufus](https://rufus.akeo.ie/?locale=zh_CN/) 工具

U盘启动做好之后，下面进入系统安装环节.

### 系统安装

#### 分区和挂载
先查看一下系统的硬盘情况
```
# lsblk # 会输出类似下面的结果(这是笔者已经分好区并挂载的情况)

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465.8G  0 disk
├─sda1   8:1    0    30G  0 part /
├─sda2   8:2    0     4G  0 part [SWAP]
└─sda3   8:3    0 431.8G  0 part /home
sdb      8:16   1    15G  0 disk
└─sdb1   8:17   1  14.6G  0 part
```
选择 `sda` 盘，开始分区
```
# cfdisk /dev/sda
```

笔者的分区情况如下, 读者根据自己的实际情况进行分区就好
```
# lsblk

sda      8:0    0 465.8G  0 disk
├─sda1   8:1    0    30G  0 part 
├─sda2   8:2    0     4G  0 part
└─sda3   8:3    0 431.8G  0 part
```
`sda1` 是根目录 `/`, `sda2` 是 `swap`, `sda3` 是 `/home`.

分区完成之后， 将分区格式化.
```
# mkfs.ext4 /dev/sda1
# mkswap /dev/sda2
# mkfx.ext4 /dev/sda3
```

分区挂载
```
# mount /dev/sda1 /mnt
# swapon /dev/sda2
# mkdir /mnt/home
# mount /dev/sda3 /mnt/home

# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465.8G  0 disk
├─sda1   8:1    0    30G  0 part /mnt
├─sda2   8:2    0     4G  0 part [SWAP]
└─sda3   8:3    0 431.8G  0 part /mnt/home
```

#### 安装系统
连接网络
```
# dhcpd # 如果是网线的，在U盘启动的时候，网络就已经连接上了
```
无线连接
```
# wifi-menu
```
测试网络
```
# ping -c3 cn.bing.com
```

配置镜像源
```
# vim /etc/pacman.d/mirrorlist
```
添加国内镜像源
```
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = http://mirrors.sohu.com/archlinux/$repo/os/$arch
```

安装系统
```
# pacstrap -i /mnt base base-devel
```
(PS: 至此，系统已经完成安装, 不过还需要一些系统初始化的配置.)

配置fstab
```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```
查看下`fstab`，看分区是否挂载上.
```
# less /mnt/etc/fstab  
```
```
# /dev/sda1
UUID=e1bb3071-2ee7-4ca5-9487-a9991b43de43       /               ext4            rw,relatime,data=ordered        0 1

# /dev/sda3
UUID=0fc36763-7d85-4d2e-8300-dca8c3e955f3       /home           ext4            rw,relatime,data=ordered        0 2

# /dev/sda2
UUID=d603302e-8f80-401d-be33-24793f6658a2       none            swap            defaults        0 0
```

切换到系统
```
# arch-chroot /mnt
```

### 配置系统
新系统没有 `vim`, 先安装
```
# pacman -S vim
```

#### 配置区域和语言

设置字符编码
```
# vim /etc/locale.gen # 去掉en_US.UTF-8,zh_CN.UTF-8注释
```

生成 `locale` 文件
```
# locale-gen
# echo LANG=en_US.UTF-8 > /etc/locale.conf
```

#### 配置时间
查看时间是否正确
```
# date
```

如果不正确， 则设置
```
# rm /etc/localtime
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# hwclock --systohc --utc
# date # 查看时间是否正确
```

#### 设置主机名和创建用户
设置主机名
```
# echo codcodog > /etc/hostname
```

为 `root` 用户修改初始化密码
```
# passwd
```

创建用户
```
# useradd -m -g users -G wheel,storage,power -s /bin/bash cryven
# passwd cryven
```

为 `cryven` 用户创建 `root` 权限
```
# nano /etc/sudoers
```
去掉注释
```
%whell ALL=(ALL) ALL
```

#### 配置网络
为笔记本安装无线连接(wifi连接)
```
# pacman -S iw wpa_supplicant dialog
```
如果是有线连接， 则不用， `dhcpd` 在系统基础包 `base` 已经安装.

#### 配置系统引导
安装grub
```
# pacman -S grub
# pacman -S os-prober # 识别多系统(例如：wins)， 可选安装
```

grub引导系统
```
# grub-install --target=i386-pc --recheck /dev/sda
```

生成grub引导文件
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

至此， 系统初始化工作，差不多完成， 卸载重启
```
# exit
# umount -R /mnt
# reboot
```
下面， 进入软件安装环节.

### 系统软件配置

Bash自动补全
```
$ sudo pacman -S bash-completion
```

安装yaourt
```
$ sudo pacman -S git

$ git clone https://aur.archlinux.org/package-query.git
$ cd package-query
$ makepkg -si

$ git clone https://aur.archlinux.org/yaourt.git
$ cd yaourt
$ makepkg -si
```
详细请戳：[Yaourt](https://archlinux.fr/yaourt-en)

安装X窗口组件
```
$ yaourt -S xorg xorg-xinit
```

安装显卡驱动
```
$ yaourt -S xf86-video-nouveau
```
驱动类型参考：[驱动](https://wiki.archlinux.org/index.php/Xorg_#Driver_installation)

安装窗口管理器
```
$ yaourt -S awesome
```
配置 `awesome`
```
$ mkdir .config/awesome
$ cp /etc/xdg/awesome/rc.lua .config/awesome/

$ vim .config/awesome/rc.lua
## titlebars_enabled = true -> titlebars_enabled = false (去掉title栏)
## terminal = "terminator --geometry=670x430" (设置启动命令端)
```

安装`terminator`
```
$ yaourt terminator
```

安装`chromium`
```
$ yaourt chromium
```

安装中文字体
```
$ yaourt wqy-microhei
```

安装输入法
```
$ yaourt -S fcitx fcitx-im fcitx-configtool fcitx-sunpinyin
```

安装`Shadows Client`
```
$ yaourt -S shadowsocks-qt5
```

安装文件浏览器
```
$ yaourt -S ranger
$ yaourt -S thunar
```

#### 配置启动文件

```
$ vim .xinitrc
```

输入如下内容：
```
# fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
exec fcitx & 2>/dev/null

# caps to ctrl
if [ -f $HOME/.Xmodmap ]; then
    /usr/bin/xmodmap $HOME/.Xmodmap
fi

# screen
bash screen 2>/dev/null

exec awesome
```

`.Xmodmap`内容如下：
```
clear lock
clear control
add control = Caps_Lock Control_L Control_R
keycode 66 = Control_L Caps_Lock NoSymbol NoSymbol
```

#### 配置Vim
安装 `Vundle`
```
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
参考：[Vundle](https://github.com/VundleVim/Vundle.vim)

下载自己的配置文件
```
$ git clone https://github.com/codcodog/Dotfiles.git
$ cp Dotfiles/.vim .
```
打开`Vim`, 输入`:PluginInstall`即可.

安装`ctrlp`插件
```
$ cd ~/.vim
$ git clone https://github.com/kien/ctrlp.vim.git bundle/ctrlp.vim
```
参考：[Ctrlp](http://kien.github.io/ctrlp.vim/#installation)

另外，`Vim` 中的 `markdown` 实时查看的那个插件还需要安装`nodejs`
```
$ yaourt -S nodejs
$ yaourt -S npm
$ sudo npm -g install instant-markdown-d
```
详细参考：[vim安装markdown插件](http://www.jianshu.com/p/24aefcd4ca93)

#### 安装SwitchyOmega插件
参考：[SwitchyOmega](https://switchyomega.com/)
