find 命令实用例子
========================
### 基础篇

首先，在主目录创建以下空文件，方便后面 `find` 命令例子演示.

```
# vim create_sample_files.sh
touch MybashProgram.sh
touch mycprogram.c
touch MyCProgram.c
touch Program.c

mkdir backup
cd backup

touch MybashProgram.sh
touch mycprogram.c
touch MyCProgram.c
touch Program.c

# chmod +x create_sample_files.sh

# ./create_sample_files.sh

# ls -R
.:
backup                  MybashProgram.sh  MyCProgram.c
create_sample_files.sh  mycprogram.c      Program.c

./backup:
MybashProgram.sh  mycprogram.c  MyCProgram.c  Program.c
```

#### 1. 根据文件名查找文件
在当前目录及其子目录下查找所有文件名为 `MyCProgram.c` 的文件.
```
# find -name "MyCProgram.c"
./backup/MyCProgram.c
./MyCProgram.c
```

#### 2. 查找时忽略大小写
```
# find -iname "MyCProgram.c"
./mycprogram.c
./backup/mycprogram.c
./backup/MyCProgram.c
./MyCProgram.c
```

#### 3. 查找时限制特定的目录层级
在根目录及其子目录下查找所有文件名为 `passwd` 的文件.
```
# find / -name passwd
./usr/share/doc/nss_ldap-253/pam.d/passwd
./usr/bin/passwd
./etc/pam.d/passwd
./etc/passwd
```

在根目录和它的一级子目录查找所有文件名为 `passwd` 的文件.
> 注意：根目录本身为1级，它的下一级子目录为2级

```
# find -maxdepth 2 -name passwd
./etc/passwd
```

在根目录和它的2级子目录查找所有文件名为 `passwd` 的文件.
```
# find / -maxdepth 3 -name passwd
./usr/bin/passwd
./etc/pam.d/passwd
./etc/passwd
```

介于根目录的2级子目录和4级子目录之间查找文件名为 `passwd` 的文件.
```
# find -mindepth 3 -maxdepth 5 -name passwd
./usr/bin/passwd
./etc/pam.d/passwd
```

#### 4. 对查找到的文件执行命令操作
`md5sum` 对查找到所有文件名为 `MyCProgram.c` 的文件进行计算. `{}` 被当前文件名替换.
```
# find -iname "MyCProgram.c" -exec md5sum {} \;
d41d8cd98f00b204e9800998ecf8427e  ./mycprogram.c
d41d8cd98f00b204e9800998ecf8427e  ./backup/mycprogram.c
d41d8cd98f00b204e9800998ecf8427e  ./backup/MyCProgram.c
d41d8cd98f00b204e9800998ecf8427e  ./MyCProgram.c
```

#### 5. 反转匹配
查找文件名不是 `MyCProgram.c` 的文件或目录.  
由于 `maxdepth` 为1，所以只在当前目录下查找.
```
# find -maxdepth 1 -not -iname "MyCProgram.c"
.
./MybashProgram.sh
./create_sample_files.sh
./backup
./Program.c
```

#### 6. 根据 inode 编号查找文件
每个文件都有一个唯一的 `inode` 编号，使用该编号能够识别该文件.  
创建两个具有相似名字的文件，其中一个文件末尾有一个空格.
```
# touch "test-file-name"

# touch "test-file-name "
[Note: There is a space at the end]

# ls -1 test*
test-file-name
test-file-name
```
从上面输出来看，没法知道那个文件最后有空格.

使用 `-i` 选项，可以查看 `inode` 编号，这两个文件的编号是不一样的.
```
# ls -i1 test*
16187429 test-file-name
16187430 test-file-name
```

通过指定 `inode` 编号来查找文件，并重命名该文件，如下所示.
```
# find -inum 16187430 -exec mv {} new-test-file-name \;

# ls -i1 *test*
16187430 new-test-file-name
16187429 test-file-name
```

如果你想对一些名称较差的文件进行某些操作，则可以使用此技术.  
例如，  
`file?.txt` 具有特殊字符. 当尝试执行 `rm file?.txt`，则以下三个文件都将被删除.
```
# ls
file1.txt  file2.txt  file?.txt
```

查看 `inode` 编号
```
# ls -i1
804178 file1.txt
804179 file2.txt
804180 file?.txt
```

利用 `inode` 编号删除具有特殊字符的文件.
```
# find -inum 804180 -exec rm {} \;

# ls
file1.txt  file2.txt
[Note: The file with name "file?.txt" is now removed]
```

#### 7. 基于文件权限查找文件
以下操作是可能的：  
- 查找匹配确切文件权限的文件
- 检查给定的权限是否匹配，而不考虑其他权限位
- 通过提供 八进制/符号表示 进行搜索

假设当前目录下包含以下文件，注意它们的权限是不同的.
```
# ls -l
total 0
-rwxrwxrwx 1 root root 0 2009-02-19 20:31 all_for_all
-rw-r--r-- 1 root root 0 2009-02-19 20:30 everybody_read
---------- 1 root root 0 2009-02-19 20:31 no_for_all
-rw------- 1 root root 0 2009-02-19 20:29 ordinary_file
-rw-r----- 1 root root 0 2009-02-19 20:27 others_can_also_read
----r----- 1 root root 0 2009-02-19 20:27 others_can_only_read
```

查找组用户具有读权限的文件，而不管该文件的其他权限如何.
```
# find . -perm -g=r -type f -exec ls -l {} \;
-rw-r--r-- 1 root root 0 2009-02-19 20:30 ./everybody_read
-rwxrwxrwx 1 root root 0 2009-02-19 20:31 ./all_for_all
----r----- 1 root root 0 2009-02-19 20:27 ./others_can_only_read
-rw-r----- 1 root root 0 2009-02-19 20:27 ./others_can_also_read
```

查找组用户只有读权限的文件.
```
# find . -perm g=r -type f -exec ls -l {} \;
----r----- 1 root root 0 2009-02-19 20:27 ./others_can_only_read
```

查找组用户只有读权限的文件，根据八进制权限位来查找.
```
# find . -perm 040 -type f -exec ls -l {} \;
----r----- 1 root root 0 2009-02-19 20:27 ./others_can_only_read
```

#### 8. 在主目录及其子目录查找所有空文件（零字节文件）
以下命令输出的大多数文件都是锁定文件和由其他应用程序创建的占位符.
```
# find ~ -empty
```

只查找主目录下的空文件
```
# find . -maxdepth 1 -empty
```

仅列出当前目录中非隐藏空白文件
```
# find . -maxdepth 1 -empty -not -name ".*"
```

#### 9. 查找前5个大文件
在当前目录及其子目录查找前5个大文件
```
# find . -type f -exec ls -s {} \; | sort -n -r | head -5
```

#### 10. 查找前5个小文件
在当前目录及其子目录查找前5个小文件
```
# find . -type f -exec ls -s {} \; | sort -n  | head -5
```
在上面的输出，很大可能只看到空文件（零字节文件）.  
用下面命令去除空文件.
```
# find . -not -empty -type f -exec ls -s {} \; | sort -n  | head -5
```

#### 11. 基于文件类型来查找文件
仅查找 `socket` 文件
```
# find . -type s
```

查找全部目录
```
# find . -type d
```

仅查找普通文件
```
# find . -type f
```

查找全部隐藏文件
```
# find . -type f -name ".*"
```

查找全部隐藏目录
```
# find -type d -name ".*"
```

#### 12. 通过与其他文件的修改时间进行比较来查找文件
显示当前目录下的文件信息
```
# ls -lrt
total 0
-rw-r----- 1 root root 0 2009-02-19 20:27 others_can_also_read
----r----- 1 root root 0 2009-02-19 20:27 others_can_only_read
-rw------- 1 root root 0 2009-02-19 20:29 ordinary_file
-rw-r--r-- 1 root root 0 2009-02-19 20:30 everybody_read
-rwxrwxrwx 1 root root 0 2009-02-19 20:31 all_for_all
---------- 1 root root 0 2009-02-19 20:31 no_for_all
```

查找在 `ordinary_file` 创建/修改 之后的文件和目录.
```
# find -newer ordinary_file
.
./everybody_read
./all_for_all
./no_for_all
```

#### 13. 根据文件大小进行查找
查找大于给定大小的文件
```
# find ~ -size +100M
```

查找小于给定大小的文件
```
# find ~ -size -100M
```

查找匹配给定大小的文件
```
# find ~ -size 100M
```

> \- 小于给定的大小，\+ 大于给定的大小，没有符号等于给定的大小.

#### 14. 为频繁查找操作创建别名
如果你发现一些非常有用的东西，那么你可以将它作为别名，方便随时执行.

经常删除文件名为 `a.out` 的文件
```
# alias rmao="find . -iname a.out -exec rm {} \;"
# rmao
```

删除由c程序生成的核心文件
```
# alias rmc="find . -iname core -exec rm {} \;"
# rmc
```

#### 15. 删除大归档文件
删除超过100M的 `*.zip` 文件.
```
# find / -type f -name *.zip -size +100M -exec rm -i {} \;"
```

创建别名
```
# alias rm100m="find / -type f -name *.tar -size +100M -exec rm -i {} \;"
# alias rm1g="find / -type f -name *.tar -size +1G -exec rm -i {} \;"
# alias rm2g="find / -type f -name *.tar -size +2G -exec rm -i {} \;"
# alias rm5g="find / -type f -name *.tar -size +5G -exec rm -i {} \;"

# rm100m
# rm1g
# rm2g
# rm5g
```

### 高级篇

#### 根据 访问/修改/更改 时间来查找文件
可以根据下面三个文件时间属性来查找文件：
- Access time 文件访问时间，每次文件被访问，该时间就进行更新.
- Modification time 文件修改时间，每次文件的内容发生了改变，这个时间就进行更新.
- Change time 文件更改时间，每次文件的 `inode` 数据发生更改之后，这个时间就进行更新.

在后面的例子中，`min` 选项和 `time` 选项之间的区别是参数.
- `min` 参数，参数的单位是分钟. 例如：min 60 = 60 分钟.
- `time` 参数，参数的单位是24小时. 例如：time 2 = 2 * 24 小时，2日.

> 在进行24小时计算的时候，小数部分会被忽略. 因此25小时取24小时，47小时取24小时，仅48小时取48小时.  

更多信息，查看 `man find` 关于 `-atime` 部分.

##### 1. 查找内容在最近1小时更新的文件
使用 `-mmin` 和 `-mtime` 选项
- `-mmin N` 文件数据在 N 分钟前最后修改的
- `-mtime N` 文件数据在 N*24 小时前最后修改的

在当前目录及其子目录查找文件内容在最近1小时得到更新的所有文件
```
# find . -mmin -60
```

在根目录及其子目录查找文件内容在最近1日得到更新的所有文件
```
# find / -mtime -1
```

##### 2. 查找1小时前访问过的文件
使用 `-amin` 和 `-atime` 选项
- `-amin N` 文件上次访问在 N 分钟前
- `-atime N` 文件上次访问在 N*24 小时前

在当前目录及其子目录查找最近1小时访问过的所有文件
```
# find -amin -60
```

在根目录及其子目录查找最近1日访问过的所有文件
```
# find / -atime -1
```

##### 3. 查找1小时前准确更改的文件
使用`-cmin` 和 `-ctime` 选项
- `-cmin N` 文件的状态在 N 分钟前最后更改
- `-ctime N` 文件的状态在 N*24 小时前最后更改

在当前目录及其子目录查找在最近1小时发生更改的所有文件
```
# find . -cmin -60
```

在根目录及其子目录查找在最近1日发生更改的所有文件
```
# find / -ctime -1
```

##### 4. 将搜索限制为普通文件
使用 `-type f` 仅查找普通文件（不查找目录）

查找最近30分钟访问的文件和目录
```
# find /etc/sysconfig -amin -30
.
./console
./network-scripts
./i18n
./rhn
./rhn/clientCaps.d
./networking
./networking/profiles
./networking/profiles/default
./networking/profiles/default/resolv.conf
./networking/profiles/default/hosts
./networking/devices
./apm-scripts
[Note: The above output contains both files and directories]
```

查找最近30分钟访问的文件
```
# find /etc/sysconfig -amin -30 -type f
./i18n
./networking/profiles/default/resolv.conf
./networking/profiles/default/hosts
[Note: The above output contains only files]
```

##### 5. 将搜索限制为不隐藏的文件
查找最近15分钟内修改的文件，仅显示不隐藏的文件
```
# find . -mmin -15 \( ! -regex ".*/\..*" \)
```

#### 比较查找文件
人类的思想可以通过引用来更好地记住事情.  
例如：  
我想要在编辑 `test` 文件后查找我编辑过的文件.  
可以通过参考其他文件修改来找到文件，如下所示.

##### 6. 查找特定文件修改后有修改的文件
```
Syntax: 
find -newer FILE
```

查找基于 `/etc/passwd` 文件修改后有修改的所有文件.  
如果你想追踪添加新用户后完成的所有活动，这很有帮助.
```
# find -newer /etc/passwd
```

##### 7. 查找特定文件修改后访问过的文件
```
Syntax: 
find -anewer FILE
```

查找基于 `/etc/hosts` 文件修改后访问过的所有文件.  
如果您还记得向 `/etc/hosts` 中添加条目，并希望查看自此以后访问过的所有文件，请使用以下命令.
```
# find -anewer /etc/hosts
```

##### 8. 查找特定文件修改后状态已更改的文件
```
Syntax: 
find -cnewer FILE
```

查找基于 `/etc/fstab` 文件修改后状态发生更改的所有文件.  
如果你还记得在 `/etc/fstab` 中添加挂载点，并想知道自那时起状态发生更改的所有文件，请使用以下命令.
```
find -cnewer /etc/fstab
```

#### 对查找到的文件执行任意操作
我们可以在 `find` 命令找到的文件执行任何操作
```
find <CONDITION to Find files> -exec <OPERATION> \;
```

`OPERATION` 可以是任何操作，例如：
- `rm` 命令，删除 `find` 命令找到的所有文件
- `mv` 命令，重命名 `find` 命令找到的所有文件
- `ls -l` 命令，获取 `find` 命令找到的所有文件的详细信息
- `md5sum` 命令，计算 `find` 命令找到的所有文件
- `wc` 命令，计算 `find` 命令找到的所有文件的总字数
- 执行 `Unix shell command`，对于 `find` 命令查找到的所有文件
- 执行 `custom shell script`，对于 `find` 命令查找到的所有文件

##### 9. 列出最近一小时内编辑的所有文件的详细信息
```
# find -mmin -60
./cron
./secure

# find -mmin -60 -exec ls -l {} \;
-rw-------  1 root root 1028 Jun 21 15:01 ./cron
-rw-------  1 root root 831752 Jun 21 15:42 ./secure
```

##### 10. 仅在当前文件系统搜索
系统管理员希望在根文件系统中搜索，但不在其他载入的分区中搜索.  
如果你安装了多个分区，并且要在 `/` 中搜索，可以执行以下操作.

在 `/` 目录下搜索 `*.log` 文件.  
如果在 `/` 下安装了多个分区，则以下命令将搜索所有这些已载入的分区.
```
# find / -name "*.log"
```

使用 `-xdev` 选项，文件搜索仅在当前文件系统进行.
> \-xdev 不要加载其他文件系统上的目录

在 `/` 目录下搜索 `*.log` 文件，并且仅在当前文件系统进行.  
如果 `/` 下安装了多个分区，则下面的命令不会搜索这些已经载入的分区.
```
# find / -xdev -name "*.log"
```

##### 11. 在同一个命令使用多个 {  }
手册说只有一个 `{  }` 的实例是可行的.  
但是你可以在同一个命令中使用多个 `{  }`，如下所示.
```
# find -name "*.txt" cp {} {}.bkup \;
```

在同一个命令使用多个 `{  }` 是可行的，但在不同的命令中使用多个 `{  }` 是不可行的.  
例如，  
你想重命名文件如下，这将不会得到预期的结果.
```
find -name "*.txt" -exec mv {} `basename {} .htm`.html \;
```

##### 12. 在多个实例中使用 {  }
你可以通过编写一个 shell 脚本来模拟它，如下所示.
```
# mv "$1" "`basename "$1" .htm`.html"
```
这些双引号是处理文件名中的空格，然后从 `find` 命令中调用该 shell 脚本.
```
find -name "*.html" -exec ./mv.sh '{}' \;
```
因此，如果你希望多次使用相同的文件名，  
那么出于原因，编写一个简单的 shell 脚本并将文件名作为参数传递是最简单的方法.

##### 13. 错误重定向到 /dev/null
重定向错误并不是一个好习惯，有经验的用户了解在终端打印错误并修复它的重要性.

特别是在 `find` 命令中错误重定向不是一个好的做法.  
但是，你如果不想看到错误并想将其重定向到 `/dev/null`，则如下所示.
```
find -name "*.txt" 2>>/dev/null
```

有时候，这可能会有帮助.  
例如，  
如果你试图从你的账户中查找 `/` 目录所有的 `*.conf` 文件，  
则可能会收到很多 `Permission denied` 的错误信息，如下所示.
```
$ find / -name "*.conf"
/sbin/generate-modprobe.conf
find: /tmp/orbit-root: Permission denied
find: /tmp/ssh-gccBMp5019: Permission denied
find: /tmp/keyring-5iqiGo: Permission denied
find: /var/log/httpd: Permission denied
find: /var/log/ppp: Permission denied
/boot/grub/grub.conf
find: /var/log/audit: Permission denied
find: /var/log/squid: Permission denied
find: /var/log/samba: Permission denied
find: /var/cache/alchemist/printconf.rpm/wm: Permission denied
[Note: There are two valid *.conf files burned in the "Permission denied" messages]
```

如果你只想查看 `find` 命令的实际输出而不是 `Permission denied` 错误信息，  
则可以将错误信息重定向到 `/dev/null`，如下所示.
```
$ find / -name "*.conf" 2>>/dev/null
/sbin/generate-modprobe.conf
/boot/grub/grub.conf
[Note: All the "Permission denied" messages are not displayed]
```

##### 14. 在文件名中用下划线替换空格
用下划线 `_` 替换所有 `*.mp3` 文件中的空格
```
$ find . -type f -iname "*.mp3" -exec rename "s/ /_/g" {} \;
```

##### 15. 同时执行两个查找命令
仅遍历文件系统一次，将 `setuid` 文件和目录列入 `/root/suid.txt`，大文件列入 `/root/big.txt`
```
# find /    \( -perm -4000 -fprintf /root/suid.txt '%#m %u %p\n' \) , \
 \( -size +100M -fprintf /root/big.txt '%-10s %p\n' \)
```

*此文为转载文章*  

原文：  
[Mommy, I found it! — 15 Practical Linux Find Command Examples](https://www.thegeekstuff.com/2009/03/15-practical-linux-find-command-examples/)  
[Daddy, I found it!, 15 Awesome Linux Find Command Examples (Part2)](https://www.thegeekstuff.com/2009/06/15-practical-unix-linux-find-command-examples-part-2/)
