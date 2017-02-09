---
title: capslock映射为ctrl
date: 2016-09-08 10:35:03
tags: 
- linux
- capslock To ctrl
categories: Linux
---
作为一个vimer，`ctrl` 使用频率是非常高的。  
所以，习惯性地配置电脑的 `capslock` 为 `ctrl`.  

需要xmodmap支持.安装xmodmap:
```
$ yaourt -S xmodmap
```
然后在用户家目录新建一个`.xmodmaprc`文件,并输入以下内容：
```
clear lock
clear control
add control = Caps_Lock Control_L Control_R
keycode 66 = Control_L Caps_Lock NoSymbol NoSymbol
```
在用户家目录执行：
```
xmodmap .xmodmaprc
```
配置文件说明：
> 把capslock映射为ctrl，shift+capslock为大写键（capslock）
> ctrl还是可以正常使用的
