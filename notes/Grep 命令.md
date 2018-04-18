Grep 命令的15个实用例子
=======================

首先创建一个示例文件 `demo_file`，方便后面 `grep` 命令演示使用.
```
$ cat demo_file
THIS LINE IS THE 1ST UPPER CASE LINE IN THIS FILE.
this line is the 1st lower case line in this file.
This Line Has All Its First Character Of The Word With Upper Case.

Two lines above this line is empty.
And this is the last line.
```

#### 1. 在单个文件中搜索给定的字符串
```
Syntax:
grep "literal_string" filename
```
`grep` 命令的基本用法：在指定的文件中搜索给定的字符串

```
$ grep "this" demo_file
this line is the 1st lower case line in this file.
Two lines above this line is empty.
And this is the last line.
```

#### 2. 在多个文件中查找给定的字符串
```
Syntax:
grep "string" FILE_PATTERN
```
这也是 `grep` 命令的基本用法. 对于这个例子，让我们将 `demo_file` 复制到 `demo_file1`.  
`grep` 的输出还将包含与指定的模式匹配的行的前面的文件名，如下所示.  
当 Linux shell 看到元字符时，它执行文件名扩展并将所有的文件作为 `grep` 的输入.
```
$ cp demo_file demo_file1

$ grep "this" demo_*
demo_file:this line is the 1st lower case line in this file.
demo_file:Two lines above this line is empty.
demo_file:And this is the last line.
demo_file1:this line is the 1st lower case line in this file.
demo_file1:Two lines above this line is empty.
demo_file1:And this is the last line.
```

#### 3. 不区分大小写的搜索
```
Syntax:
grep -i "string" FILE
```
这也是 `grep` 命令的基本用法.  
这会不区分字母的大小写来搜索给定的字符串，所以，它能够匹配 `the`, `THE`, `The` 这类的单词，如下所示.
```
$ grep -i "the" demo_file
THIS LINE IS THE 1ST UPPER CASE LINE IN THIS FILE.
this line is the 1st lower case line in this file.
This Line Has All Its First Character Of The Word With Upper Case.
And this is the last line.
```

#### 4. 在文件中匹配正则表达式
```
Syntax:
grep "REGEX" filename
```
如果你熟悉正则表达式，这将是一个非常强大的功能.  
在下面的例子，搜索所有以 `lines` 开头并以 `empty` 结尾的模式. 即在 `demo_file` 搜索 `lines[anythin in-between]empty`
```
$ grep "lines.*empty" demo_file
Two lines above this line is empty.
```

一个正则表达式可能跟着以下几个操作符：
- `？` 最多匹配一次
- `*` 匹配零次或多次
- `+` 匹配一次或多次
- `{n}` 匹配n次
- `{n,}` 匹配n次或多次
- `{,m}` 最多匹配m次
- `{n,m}` 最少匹配n次，最多匹配m次

#### 5. 查找单词
如果你想查找一个单词，而不是一个子字符串，使用 `-w` 选项.

下面的例子是一个普通的 `grep` 查找 `is` 字符.   
当查找 `is` 字符，没有任何选项，它将会匹配 `is`, `his`, `this` 以及任何字符包含 `is` 字串的.
```
$ grep -i "is" demo_file
THIS LINE IS THE 1ST UPPER CASE LINE IN THIS FILE.
this line is the 1st lower case line in this file.
This Line Has All Its First Character Of The Word With Upper Case.
Two lines above this line is empty.
And this is the last line.
```

下面的例子是单词查找 `is`.  
注意，这里匹配输出将不会包含 `This Line Has All Its First Character Of The Word With Upper Case.` 这一行，即使 `is` 包含在 `This` 中.  
因为，下面例子是仅查找单词 `is` 而不是 `This`.
```
$ grep -iw "is" demo_file
THIS LINE IS THE 1ST UPPER CASE LINE IN THIS FILE.
this line is the 1st lower case line in this file.
Two lines above this line is empty.
And this is the last line.
```

#### 6. 显示匹配 之前/之后/周围 的行
在大文件中查找的时候，在匹配后查看一些行可能会很有用.  
如果 `grep` 能够查看匹配 之后/之前/周围 的行，你会感到非常方便.

先创建 `demo_text` 演示文件
```
$ cat demo_text
4. Vim Word Navigation

You may want to do several navigation in relation to the words, such as:

 * e - go to the end of the current word.
 * E - go to the end of the current WORD.
 * b - go to the previous (before) word.
 * B - go to the previous (before) WORD.
 * w - go to the next word.
 * W - go to the next WORD.

WORD - WORD consists of a sequence of non-blank characters, separated with white space.
word - word consists of a sequence of letters, digits and underscores.

Example to show the difference between WORD and word

 * 192.168.1.1 - single WORD
 * 192.168.1.1 - seven words.
```

##### 显示匹配后 N 行
`-A` 选项，打印匹配后的 N 行
```
Syntax:
grep -A <N> "string" FILENAME
```

打印匹配后3行
```
$ grep -A 3 -i "example" demo_text
Example to show the difference between WORD and word

* 192.168.1.1 - single WORD
* 192.168.1.1 - seven words.
```

##### 显示匹配前 N 行
`-B` 选项，打印匹配前的 N 行
```
Syntax:
grep -B <N> "string" FILENAME
```

打印匹配前的2行
```
$ grep -B 2 "single WORD" demo_text
Example to show the difference between WORD and word

* 192.168.1.1 - single WORD
```

##### 显示匹配周围的 N 行
`-C` 选项，打印匹配周围的 N 行

打印匹配周围的2行
```
$ grep -C 2 "Example" demo_text
word - word consists of a sequence of letters, digits and underscores.

Example to show the difference between WORD and word

* 192.168.1.1 - single WORD
```

#### 7. 高亮搜索
`grep` 会根据提供的 模式/字符 打印出匹配的行，但如果想高亮显示，则需要设置 `GREP_OPTIONS` 环境变量.

```
$ export GREP_OPTIONS='--color=auto' GREP_COLOR='100;8'

$ grep this demo_file
<b>this</b> line is the 1st lower case line in this file.
Two lines above <b>this</b> line is empty.
And <b>this</b> is the last line.
```

#### 8. 递归搜索所有文件
当你想搜索当前目录及其子目录下的所有文件的时候，`-r` 选项就是你需要使用的.

在当前目录及其子目录的所有文件中搜索字符 `ramesh`
```
$ grep -r "ramesh" *
```

#### 9. 反转匹配
当你想显示那些没有匹配给出的 字符/模式 的行时，`-v` 选项就是你需要的.

显示所有没有匹配 `go` 的行
```
$ grep -v "go" demo_text
4. Vim Word Navigation

You may want to do several navigation in relation to the words, such as:

WORD - WORD consists of a sequence of non-blank characters, separated with white space.
word - word consists of a sequence of letters, digits and underscores.

Example to show the difference between WORD and word

* 192.168.1.1 - single WORD
* 192.168.1.1 - seven words.
```

#### 10. 显示不符合所有给定模式的行
```
Syntax:
grep -v -e "pattern" -e "pattern"
```

```
$ cat test-file.txt
a
b
c
d

$ grep -v -e "a" -e "b" -e "c" test-file.txt
d
```

#### 11. 显示匹配的数量
当想知道有多少行匹配给出的 字符/模式 的时候，使用 `-c` 选项.
```
Syntax:
grep -c "pattern" filename
```

有多少行匹配
```
$ grep -c this demo_file
3
```

有多少行没有匹配
```
$ grep -v -c this demo_file
4
```

#### 12. 仅显示匹配模式的文件名
当你想 `grep` 只输出给定模式匹配的文件名的时候，使用 `-l` 选项.

```
$ grep -l this demo_*
demo_file
demo_file1
```

#### 13. 仅显示匹配的字符串
`grep` 默认输出给定 字符/模式 匹配的行，如果你只想输出匹配模式的字符串，则使用 `-o` 选项.

当你直接提供字符串的时候，它可能没那么有用.  
但当你提供模式并试图查看它匹配的内容的时候，它会变得非常有用.
```
$ grep -o "is.*line" demo_file
is line is the 1st lower case line
is line
is is the last line
```

#### 14. 在行中显示匹配的位置
当你想 `grep` 显示它与文件中的模式相匹配的位置时，使用以下选项.
```
Syntax:
grep -o -b "pattern" file
```

```
$ cat temp-file.txt
12345
12345

$ grep -o -b "3" temp-file.txt
2:3
8:3
```
> 注意：上面 `grep` 命令的输出不是行中的位置，它是整个文件的字节偏移量.

#### 15. 输出时显示行号
用匹配的行显示文件的行号
```
$ grep -n "go" demo_text
5: * e - go to the end of the current word.
6: * E - go to the end of the current WORD.
7: * b - go to the previous (before) word.
8: * B - go to the previous (before) WORD.
9: * w - go to the next word.
10: * W - go to the next WORD.
```

*此文为转载文章*  
原文：[15 Practical Grep Command Examples In Linux / UNIX](https://www.thegeekstuff.com/2009/03/15-practical-unix-grep-command-examples/)
