test, [, [[
===========

### 定义

#### test
`test` 是 `shell` 的一个内置命令，主要用来检查文件类型和比较值.  
```bash
test -f "$filename" || printf '文件不存在或者不是普通文件: %s\n' "$filename" >&2
```

#### [  ]
``[`` 是 `test` 的一个代名词（简写），用法同 `test` 一样，即 `test expr` 等同于 `[ expr ]`
```bash
[ -f "$filename" ] || printf '文件不存在或者不是普通文件: %s\n' "$filename" >&2
```
> [ 最后一个参数必须是 ]，以匹配开头的 [ ,  而且 [  ] 两边需要有空格分隔

#### [[  ]]
``[[`` 是 `[` 的改进版，更加方便使用，可以理解为现代版 `test`
```bash
[[ -f "$filename" ]] || printf '文件不存在或者不是普通文件: %s\n' "$filename" >&2
```
> [[ 是 `shell` 的一个关键字，[[  ]] 两边需要空格分隔

### 区别
| Feature | New test [[ | Old test [ | Example |
|-------|:-----------:|:----------:|-------|
| 字符串 | > | \\> | [[ a > b ]] \|\| echo "a does not come after b" |
| 字符串 | `<` | \\`<` | [[ a `<` b ]] \|\| echo "a does not come after b" |
| 字符串 | = (or ==) | = | [[ a = a ]] && echo "a equals a" |
| 字符串 | != | != | [[ a != b ]] && echo "a is not equal to b" |
| 整数 | -gt | -gt | [[ 5 -gt 10 ]] \|\| echo "5 is not bigger than 10" |
| 整数 | -lt | -lt | [[ 8 -lt 9 ]] && echo "8 is less than 9" |
| 整数 | -ge | -ge | [[ 3 -ge 3 ]] && echo "3 is greater than or equal to 3" |
| 整数 | -le | -le | [[ 3 -le 8 ]] && echo "3 is less than or equal to 8" |
| 整数 | -eq | -eq | [[ 5 -eq 05 ]] && echo "5 equals 05" |
| 整数 | -ne | -ne | [[ 6 -ne 20 ]] && echo "6 is not equal to 20" |
| 条件 | && | -a | [[ -n $var && -f $var ]] && echo "$var is a file" |
| 条件 | \|\| | -o | [[ -b $var \|\| -c $var ]] && echo "$var is a device" |
| 分组 | (...) | \\(...\\) | [[ $var = img* && ($var = *.png \|\| $var = *.jpg) ]] && echo "$var starts with img and ends with .jpg or .png" |
| 模式匹配 | = (or ==) | not available | [[ $name = a\* ]] \|\| echo "name does not start with an 'a': $name"  |
| 常规正则匹配 | =~ | not available | [[ $(date) =~ ^Fri\\...\\ 13 ]] && echo "It's Friday the 13th!" |

`[` 条件测试的时候，除了用 `-a` 和 `-o`, 还可以这样用
```bash
if [ "$a" = a ] && [ "$b" = b ]; then ...  

# 等价于
if [ "$a" = a -a "$b" = b ]; then ...
```

```bash
if [ "$a" = a ] || { [ "$b" = b ] && [ "$c" = c ]; }; then ...
```

在使用 `[[` 的时候，没有单词拆分(WordSplitting) 和 glob 扩展.
```bash
file="file name"
[[ -f $file ]] && echo "$file is a regular file."
```
`$file` 包含空格而且没有被引号括起来，在 `[[` 是可以正常使用的，而在 `[` 则必须被双引号引起来才行
```bash
file="file name"
[ -f "$file" ] && echo "$file is a regular file."
```

一般 `[[` 是用来进行字符串和文件的测试，如果想比较数字的话，则推荐使用算术表达式
```bash
i=0
while ((i < 10)); do ...
```

### 使用哪种
`test` 实现了旧的可移植的语法.  

在大多数 `shell` (最老的 Bourne Shell 除外), `[` 都是 `test` 的代名词（简写）.  
虽然所有现代的 `shell` 都有内置的 `[` 的实现，但通常还有该名称的外部的可执行文件，例如：`/bin/[`. 如果想要可移植的代码，则不要使用这些外部扩展命令.

`[[` 是 `[` 的一个新的改进版本，并且是一个关键字而不是程序.  
`[[` 可以被 `KornShell`, `Zsh`  和 `Bash`(从版本2.03开始) 理解，但不能被其他 POSIX shell(例如：posh, yash, dash) 实现或 BourneShell 理解.

如果考虑对 POSIX 或 BourneShell 的可移植性和一致性，则使用旧语法的 `[`  
如果是使用 `Bash`, `Zsh`, `KornShell` 的话，则使用新语法的 `[[`，这样会更加灵活方便.

**参考文章**  
[What is the difference between test, \[ and \[\[ ?](http://mywiki.wooledge.org/BashFAQ/031)
