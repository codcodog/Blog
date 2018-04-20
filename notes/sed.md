sed
====
`sed` 是一个「流编辑器」，在 Unix(Linux) 用于修改文件. 

无论你什么时候想自动修改文件，`sed` 都可以方便地实现.  
大部分人都没有了解到 `sed` 的强大，只是简单地用来替换文本.  
除了用 `sed` 来替换文本，你还可以做很多事情. 下面用一些示例来描述 `sed` 的功能.

默认将下面的文本作为输入
```
>cat file.txt
unix is great os. unix is opensource. unix is free os.
learn operating system.
unixlinux which one you choose.
```

#### 1. 替换字符
`sed` 主要用于替换文件中的文本.  
如下所示，替换 `unix` 为 `linux`
```
>sed 's/unix/linux/' file.txt
linux is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
```
`s` 指替换操作，`/` 指分隔符，`unix` 是搜索模式，`linux` 是替换字符串.

`sed` 默认会替换每行中第一次出现匹配模式的字符串，而不会替换该行中出现的第二个，第三个，...

#### 2. 替换匹配模式的第N个字符串
使用 `/1`, `/2` 这样的标志来替换一行中出现的第一个，第二个匹配模式的字符串.  

用 `linux` 替换第二个出现的 `unix`
```
>sed 's/unix/linux/2' file.txt
unix is great os. linux is opensource. unix is free os.
learn operating system.
unixlinux which one you choose.
```

#### 3. 替换匹配模式的全部字符串
`/g` 标志替换匹配模式的全部字符串
```
>sed 's/unix/linux/g' file.txt
linux is great os. linux is opensource. linux is free os.
learn operating system.
linuxlinux which one you choose.
```

#### 4. 从第N个开始匹配的字符串开始替换全部匹配的字符串
从第三个匹配的字符串开始，替换全部的 `unix` 为 `linux`
```
>sed 's/unix/linux/3g' file.txt
unix is great os. unix is opensource. linux is free os.
learn operating system.
unixlinux which one you choose.
```

#### 5. 修改 / 分隔符
可以使用 `/` 之外的任何分隔符

修改 web url 为另外一个 url
```
>sed 's/http:\/\//www/' file.txt
```
如上面例子，需要转义 `/` 以区分分隔符.  
另外，可以通过修改分隔符来减少这种额外的操作，并且看起来也更简洁.
```
>sed 's_http://_www_' file.txt
>sed 's|http://|www|' file.txt
```

#### 6. & 作为匹配的字符串
可能某些情况下，想要搜索该模式并通过向其添加一些额外的字符来替换该模式，则 `&` 字符就派上用场了.
```
>sed 's/unix/{&}/' file.txt
{unix} is great os. unix is opensource. unix is free os.
learn operating system.
{unix}linux which one you choose.

>sed 's/unix/{&&}/' file.txt
{unixunix} is great os. unix is opensource. unix is free os.
learn operating system.
{unixunix}linux which one you choose.
```

#### 7. 使用 \1, \2, ..., \9 等等
模式中指定的第一对括号表示 `\1`，第二对括号表示 `\2` 等等.

用两倍的 `unix` 即 `unixunix` 来替换 `unix`
```
>sed 's/\(unix\)/\1\1/' file.txt
unixunix is great os. unix is opensource. unix is free os.
learn operating system.
unixunixlinux which one you choose.
```
> 括号应该使用 \ 转义

替换 `unixlinux` 为 `linuxunix`
```
>sed 's/\(unix\)\(linux\)/\2\1/' file.txt
unix is great os. unix is opensource. unix is free os.
learn operating system.
linuxunix which one you choose.
```

切换前三个字符
```
>sed 's/^\(.\)\(.\)\(.\)/\3\2\1/' file.txt
inux is great os. unix is opensource. unix is free os.
aelrn operating system.
inuxlinux which one you choose.
```

#### 8. 复制替换的行
`/p` 标志在终端上打印两次被替换的行.  
如果一行没有匹配搜索模式的字符并且没有被替换，那么 `/p` 只打印一行.
```
>sed 's/unix/linux/p' file.txt
linux is great os. unix is opensource. unix is free os.
linux is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
linuxlinux which one you choose.
```

#### 9. 只打印替换的行
`-n` 选项和 `/p` 标志一起使用，只显示替换的行.  
`-n` 选项禁止 `/p` 标志生成的重复行，并只打印一次被替换的行.
```
>sed -n 's/unix/linux/p' file.txt
linux is great os. unix is opensource. unix is free os.
linuxlinux which one you choose.
```

> 如果只用 -n 选项， sed 将不会打印任何东西.

#### 10. 执行多个 sed 命令
将一个 `sed` 的输出作为另外一个 `sed` 的输入
```
>sed 's/unix/linux/' file.txt| sed 's/os/system/'
linux is great system. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you chosysteme.
```

`-e` 选项可以在单个 `sed` 命令运行多个 `sed` 脚本.
```
>sed -e 's/unix/linux/' -e 's/os/system/' file.txt
linux is great system. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you chosysteme.
```

#### 11. 在特定的行替换文本
只在第3行替换文本
```
>sed '3 s/unix/linux/' file.txt
unix is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
```

#### 12. 在某个范围内的行替换文本
在第1行到第3行替换文本
```
>sed '1,3 s/unix/linux/' file.txt
linux is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
```

从第2行到最后一行替换文本
```
>sed '2,$ s/unix/linux/' file.txt
linux is great os. unix is opensource. unix is free os.
learn operating system.
linuxlinux which one you choose.
```

#### 13. 在匹配模式的行替换文本
可以指定一个模式来匹配某些行.  
如果某行匹配了模式，`sed` 则会在该行进行查找和替换操作.

查找某行匹配模式 `linux`，然后在该行将 `unix` 替换为 `centos`
```
>sed '/linux/ s/unix/centos/' file.txt
unix is great os. unix is opensource. unix is free os.
learn operating system.
centoslinux which one you choose.
```

#### 14. 删除行
可以指定某行或某个范围的行进行删除
```
>sed '2 d' file.txt
>sed '5,$ d' file.txt
```

#### 15. 复制行
使用 `sed` 命令打印两次文件的每一行
```
>sed 'p' file.txt
```

#### 16. sed 类似 grep
查找模式 `unix` 并打印出这些行
```
>grep 'unix' file.txt
>sed -n '/unix/ p' file.txt
```

查找不匹配模式 `unix` 并打印出这些行
```
>grep -v 'unix' file.txt
>sed -n '/unix/ !p' file.txt
```
> ! 反转匹配

#### 17. 在匹配的行后添加一行
`sed` 的 `a` 命令可以在找到模式匹配后添加新行.
```
>sed '/unix/ a "Add a new line"' file.txt
unix is great os. unix is opensource. unix is free os.
"Add a new line"
learn operating system.
unixlinux which one you choose.
"Add a new line"
```

#### 18. 在匹配的行前添加一行
`sed` 的 `i` 命令可以在找到模式匹配前添加新行.
```
>sed '/unix/ i "Add a new line"' file.txt
"Add a new line"
unix is great os. unix is opensource. unix is free os.
learn operating system.
"Add a new line"
unixlinux which one you choose.
```

#### 19. 修改一行
`sed` 的 `c` 命令可以修改模式匹配的行.
```
>sed '/unix/ c "Change line"' file.txt
"Change line"
learn operating system.
"Change line"
```

#### 20. 像 tr 命令一样转换
通过使用 变换 `y` 选项，`sed` 可以把小写字母转换为大写字母.

把字母 `ul` 转换为大写 `UL`
```
>sed 'y/ul/UL/' file.txt
Unix is great os. Unix is opensoUrce. Unix is free os.
Learn operating system.
UnixLinUx which one yoU choose.
```

*此文为转载文章*  
原文：[Sed Command in Unix and Linux Examples](http://www.folkstalk.com/2012/01/sed-command-in-unix-examples.html)
