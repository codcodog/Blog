---
title: Shell Scripting part2
date: 2016-09-19 21:34:10
tags: 
- linux
- shell
categories: Linux
---

#### shell之正则表达式 ####
glob: 文件名称通配  
`*`: 任意长度任意字符.  
`?`  
`[]`  
`[^]`  

grep: Global RE(Regular Expression) Printing  
文本过滤工具，能够实现根据指定的“模式（pattern）”逐行搜索文件内容，并将匹配到的行显示出来。  

RE:  
&nbsp;&nbsp;Basic RE.  
&nbsp;&nbsp;Extended RE.  

`grep [option] ... 'PATTERn' FILE`  
`grep -v`,`grep --color=auto`  

基本正则表达式元字符：  
`.`: 匹配任意单个字符.  
`[]`: 匹配指定范围内的任意单个字符.  
`[^]`: 匹配指定范围外的任意单个字符.  

次数匹配:  
`*`: 匹配其前的字符0,1或多次.(贪婪模式),例如：  

    ab*c: abc, abbc, ac, abdc(不匹配), abbbbbbbc
    
`?`: 匹配其前的字符0或1次.例如：  

    ab?c: ac, abc, adc(不匹配), abbc(不匹配)
    
`\{m,n\}`: m-n次  
`\{m,\}`: 至少m次  
`\{0,n\}`: 至多n次  
`\{m\}`: m次  

`[:space:]` 匹配空白字符，空格或者tab键.  
`[:punct:]` 标点符号.  

锚定符：  
单词锚定  
`\<`: 锚顶词尾.  
`\>`: 锚顶词首.  

行首锚定  
`^`: 行首锚定.  
`$`: 行尾锚定.  

`.*`: 任意长度的任意字符.  

分组  
`\(\)`, `x\(ab\)*y`.  
前向引用, \1, \2, \3.  
```
He love his lover.
She like her liker.
He love his liker.
She likes her lover.
```
匹配：如果找到`love`,则匹配`lover`.  
``\(l..e\).*\1r``: love...lover,like...liker.  

grep 选项  
`-v`: 显示不被模式匹配的行.  
`-i`: 不区分字符大小写.  
`-o`: 只显示匹配的字符串.  
`-A #`: 显示匹配到的后#行  
`-B #`:
`-C #`: 前后#行.  
`grep -E`: 使用扩展正则表达式.  

#### egrep及扩展正则表达式 ####
正则表达式：  
Basic REGEXP: 基本  
Extended REGEXP: 扩展  

扩展正则表达式:  
字符匹配：  
`.`, `[]`, `[^]`  
次数匹配：  
`*`, `?`, `+`, `{m,n}`  
位置锚定跟基本一样.  
分组：`()`  

或者：`|`, `C|cat` => `C或者cat`, `(C|c)at` => `Cat或者cat`  

`egrep = grep -E`  

IPv4:  
5类： A,B,C,D,E  
A: 1-127  
B: 128-191  
C: 192-223  

#### linux用户权限和文件权限深入解析 ####
perm的指定方法：  
1. 八进制  
```
    chmod 660 /tmp/a.txt, rw-rw----  
```
2. 类别范围  
```    
    chmod u=rx FILE ...
    g=
    ug=
    o=
    a=
```
3. 只操作某类用户的某位权限，利用`+/-`  
```    
    u-
    o+
```
改变文件属主属组，只有管理员才有权限.  
`chown [option] USENAME FILE ...`  
`chgrp [option] GRPNAME FILE ...`  
    
    chown USERNAME:GRPNAME FILE ...
    chown USERNAME.GRPNAME FILE ...

用户切换  
`su USERNAME`: 非登陆式切换.  
`su - USERNAME`: 登陆式切换.  
`su -l USERNAME`: 登陆式切换.  
`su USERNAME -c COMMAND`  

显示信息：  
`echo`, `printf`  
`echo -n, -e`

#### shell脚本编程 ####
bash算术运算符：  
`+, -, *, /, %`  

`let`, 算术运算, `let SUM=2+3`  
`expr 2 + 3`  
`$[arithexpr]`, `SUM=$[2+3]`  
`SUM=$((2+3))`  
`$()` 是命令替换.  
`$(())` 是算术运算.  

bash只支持整除  
`1/2=0`, `4/3=1`  
`bc`, `利用bc scale可以实现浮点运算`, 例如：  
    
    echo 'scale=6;3/2' | bc
    1.500000

逻辑运算符：  
按位进行逻辑  
逻辑关系运算  

(整数)比较运算：  
`test expression`  
`[ expresion ]` 注意中括号两端有空白字符.  
`[[ expresion ]]`  

`NUM1 -eq NUM2`, 等于  
`NUM1 -ne NUM2`, 不等于  
`NUM1 -gt NUM2`, 大于  
`NUM1 -lt NUM2`, 小于  
`NUM1 -ge NUM2`, 大于等于  
`NUM1 -le NUM2`, 小于等于  
`test 2 -gt 3`  

字符串测试：  
`'STRING1' == 'STRING2'`, 如果是变量，则字符串用双引号而不是单引号.  
`!=, <>` 表示两个字符串不相等.  
`-n "$A"`
`-z "$A"` 测试字符串是否为空.  

退出脚本：  
`exit NUM` 失败退出.  
`exit 0` 成功退出.  
