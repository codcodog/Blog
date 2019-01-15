问题集锦
========

1. makefile 中 .PYONY 的目的是什么  
    默认情况下，`Makefile` 目标是「文件目标」- 它们用于从其他文件构建文件. 假设它的目标是一个文件，这使得编写 `Makefile` 相对容易.
    ```
    foo: bar
        create_one_from_the_other foo bar
    ```
    但是，有时你希望 `Makefile` 运行一些命令而不是构建一个在文件系统中的实际文件. 例如常用的目标文件：`clean`, `all`.  
    但有时情况并非如此，你可能在主目录中有一个名为 `clean` 的文件，这时候 `Make` 会被混淆，因为默认情况下，`clean` 目标将与此文件关联，而 `Make` 只会在这个文件根据它的依赖发现不是最新的时候才会执行.  

    这些特殊目标称为「虚假目标」，你可以明确告诉 `Make` 它们与文件无关.
    ```
    .PHONY: clean
    clean:
        rm -rf *.o
    ```
    现在 `make clean` 将按预期运行，即使你有一个名为 `clean` 的文件.  

    就 `Make` 而言，「虚假目标」是一个总是过时的目标，因此无论何时  `make <pyony_target>`, 它都会运行，独立于文件系统的状态.  
    一些常用的「虚假目标」：`all`, `install`, `clean`, `distclean`, `TAGS`, `info`, `check`.

2. GNU Makefile 变量赋值 =, ?=, := 和 += 之间有什么区别  
    #### Lazy Set
    ```
    VARIABLE = value
    ```
    变量的正常设置 - 当使用变量时，变量会「递归扩展」，而不是在声明变量的时候扩展.  
    #### Immediate Set
    ```
    VARIABLE := value
    ```
    通过简单扩展内部值来设置变量 - 其中的值在声明时扩展.（不会「递归扩展」）.  
    #### Set If Absent
    ```
    VARIABLE ?= value
    ```
    如果变量没有定义则设置变量.
    #### Append
    ```
    VARIABLE += value
    ```
    将提供的值附加到现有的值（如果变量不存在，则设置为该值）.

3. 从命令行传递其他变量给 make  
    你有几个选择可以从 `makefile` 外部设置变量.  
    1. **环境变量** - 每一个环境变量都转化为具有相同名称和值的 `makefile` 变量.  

    你可能还想设置 `-e` 选项（或者 `--environments-override`），并且你的环境变量会覆盖 `makefile` 中的赋值（除非这些赋值本身使用 `override`）
    指令. 但是不推荐使用它.  
    而是推荐使用更好更灵活的 `?=` 条件赋值（只有在尚没定义变量时才有效）.
    ```
    FOO ?= default_value_if_not_set_in_environment
    ```
    注意，某些变量不是从环境变量继承的
    - `MAKE` 从脚本的名字得到的
    - `SHELL` 要么设置在 `makefile` 中，要么默认为 `/bin/sh`

    2. **从命令行** - `make` 可以将变量赋值作为命令行的一部分，与目标混合.
    ```
    make target FOO=bar
    ```
    但是，除非在赋值中使用 `override` 指令，否则忽略 `makefile` 中 `FOO` 变量的所有值.

    3. **从父 make 导出** - 如果从 `makefile` 调用 `make`, 通常不应该这样显式地写这样的变量赋值
    ```
    # Don't do this!
    target:
        $(MAKE) -C target CC=$(CC) CFLAGS=$(CFLAGS)
    ```
    相反，更好的方法是导出这些变量.  
    导出变量使其进入每个 `shell` 调用的环境，并且使用这些命令将会调用这些环境变量，如上所述.
    ```
    # Do like this
    CFLAGS=-g
    export CFLAGS
    target:
        $(MAKE) -C target
    ```
    你也可以导出全部变量，直接使用 `export` 不带参数.

4. makefile 中 $@ 和 $^ 是什么意思  
    `$@` 是要生成的目标文件的名称，而 `$^` 是第一个先决条件（通常是源文件）. 例如：
    ```
    all: library.cpp main.cpp
    ```
    `$@` 相当于 `all`  
    `$<` 相当于 `library.cpp`  
    `$^` 相当于 `library.cpp main.cpp`

5. makefile:4: \*** missing separator. Stop  
    `makefile` 与制表符有非常愚蠢的关系.  
    每个规则的所有操作都由制表符标识，而不是4个空格，只有 `tab` 才能标识为制表符.  
    去检查是否使用制表符，使用 `cat -e -t -v makefile_name`.  
    它显示了带有 `^I` 的 `tab` 和带有 `$` 的行结尾，两者确保了依赖关系正确结束以及制表符标识规则的操作以使它们易于识别为 `make` 实用程序起到至关重要的作用.
    ```
    Kaizen ~/so_test $ cat -e -t -v  mk.t
    all:ll$      ## here the $ is end of line ...
    $
    ll:ll.c   $
    ^Igcc  -c  -Wall -Werror -02 c.c ll.c  -o  ll  $@  $<$
    ## the ^I above means a tab was there before the action part, so this line is ok .
    $
    clean :$
        \rm -fr ll$
    ## see here there is no ^I which means , tab is not present ....
    ## in this case you need to open the file again and edit/ensure a tab
    ## starts the action part
    ```

6. 什么是 Makefile.am 和 Makefile.in  
    `Makefile.am` 是程序员定义的文件，由 `automake` 用于生成 `Makefile.in` 文件（`.am` 代表 `automake`）.  
    通常在源码包中可以看到 `configure` 脚本使用 `Makefile.in` 生成 `Makefile`.

    `configure` 脚本本身是从名为 `configure.ac` 或 `configure.in`（不建议使用）的程序员定义的文件生成的.  
    我更喜欢 `.ac` （用于 `autoconf`），因为它将与生成的 `Makefile.in` 文件区分开来，这样我就可以使用 `make dist-clean` 等规则来运行
    `rm -f *.in`.  
    由于它是生成的文件，因此通常不会存储在版本系统（Git，Svn）中，而是存储在 `.ac` 文件中.

7. 如何在 makefile 中编写 cd 命令  
    它实际上是执行命令，将目录更改为 `some_directory`, 但是，这是在子进程 `shell` 中执行的，并且既不影响 `make` 也不影响你正在使用的 `shell`.

    如果你想在 `some_directory` 中执行更多任务，则需要添加分号并附加其他命令.  
    请注意，你不能使用换行符，因为它们被 `make` 解释为规则的结尾，因此为清晰起见而使用的任何换行都需要通过反斜杠进行转义.
    ```
    all:
        cd some_dir; echo "I'm in some_dir"; \
        gcc -Wall -o myTest myTest.c
    ```

    一个常见的用法是在子目录中调用 `make`. 这有一个命令行选项，所以你不必自己调用 `cd`.
    ```
    all:
        $(MAKE) -C some_dir all
    ```
    将更改为 `some_dir` 目录并执行目标为 `all` 的 `Makefile`.  
    作为最佳实践，使用 `$(MAKE)` 而不是直接调用 `make`, 因为它会注意调用正确的 `make` 实例（例如，如果你为构建环境使用特殊的 `make` 版本），并提供使用某些开关（例如，`-t`）运行时略有不同的行为.

8. 如何在 makefile 中打印变量  
    你可以使用此方法（使用名为 `var` 的变量）在读取 `makefile` 时打印出变量.
    ```
    $(info $$var is [${var}])
    ```
    你可以将此构造添加到任何配方中，以查看将传递给 `shell` 的内容
    ```
    .PYONY: all
    all: ; $(info $$var is [${var}])echo Hello World
    ```
    现在，这里发生的是将整个配方（`$(info $$var is [${var}])echo Hello World`）存储为单个递归扩展变量. 当 `make` 决定运行配方时（例如，当你告诉它构建 `all` 时），它会扩展变量，然后将每个结果行分别传递给 `shell`.

    所以，令人懊恼的细节：  
    - 扩展 `$(info $$var is [${var}])echo Hello World`
    - 要做到这点，首先扩展 `$(info $$var is [${var}])`
        + `$$` 扩展成 `$`
        + `${var}` 扩展成 `:)` （取决你变量本身的值）
        + 副作用是 `$var is [:)]` 出现在标准输出（屏幕）上
        + `$(info ...)` 扩展成空
    - `Make` 还剩下 `echo Hello World`
        + `Make` 首先输出 `echo Hello World` 到标准输出（屏幕），让你知道它要求 `shell` 做什么
    - `shell` 输出 `Hello World` 到标准输出（屏幕）上

9. 如何在 makefile 中编写循环  
   假设你使用的是 `UNIX` 类型平台，如果运行 `./a.out`.
   ```
   for number in 1 2 3 4 ; do \
       ./a.out $$number ; \
   done
   ```
   测试如下
   ```
   target:
       for number in 1 2 3 4 ; do \
           echo $$number ; \
       done
   ```
   输出
   ```
   1
   2
   3
   4
   ```
   对于更大的范围，使用
   ```
   target:
       number=1 ; while [[ $$number -le 10 ]] ; do \
           echo $$number ; \
           ((number = number + 1)) ; \
       done
   ```
   嵌套循环
   ```
   target:
       num1=1 ; while [[ $$num1 -le 4 ]] ; do \
           num2=1 ; while [[ $$num2 -le 3 ]] ; do \
               echo $$num1 $$num2 ; \
               ((num2 = num2 + 1)) ; \
               done ; \
           ((num1 = num1 + 1)) ; \
       done
   ```
   输出
   ```
   1 1
   1 2
   1 3
   2 1
   2 2
   2 3
   3 1
   3 2
   3 3
   4 1
   4 2
   4 3
   ```

10. 在 Unix 中，我可以在一个目录中运行 make 而不先 cd 到那个目录吗  
    `make -C /path/to/dir`

11. 如何将命令的输出分配给 Makefile 变量  
    使用 `Make` 内建函数 `shell`, 例如：`MY_VAR = $(shell echo whatever)`
    ```
    $ make
    MY_VAR IS whatever
    ```
    ```
    $ cat Makefile
    MY_VAR := $(shell echo whatever)

    all:
        @echo MY_VAR IS $(MY_VAR)
    ```

12. 如何配置 makefile 为调试和发布版本  
    使用特定于目标的变量值，例如
    ```
    CXXFLAGS = -g3 -gdwarf2
    CCFLAGS = -g3 -gdwarf2

    all: executable

    debug: CXXFLAGS += -DDEBUG -g
    debug: CCFLAGS += -DDEBUG -g
    debug: executable

    executable: CommandParser.tab.o CommandParser.yy.o Command.o
        $(CXX) -o output CommandParser.yy.o CommandParser.tab.o Command.o -lfl

    CommandParser.yy.o: CommandParser.l
        flex -o CommandParser.yy.c CommandParser.l
        $(CC) -c CommandParser.yy.c
    ```
    请记住使用 `$(CXX)` 或 `$(CC)` 作为编译命令.  
    然后，`make debug` 将会有额外的标识，如：`-DDEBUG` 和 `-g`, 而 `make` 不会有.  

13. 如何获取 Makefile 的当前相对目录  
    你可以使用 `shell` 函数：`current_dir = $(shell pwd)`.  
    或者你需要绝对路径，可以 `shell` 结合 `notdir` 使用, `current_dir = $(notdir $(shell pwd))`.  
    上面的方法，仅 `make` 在 `Makefile` 当前目录执行有效.

    **注意：** 这将返回当前工作目录，而不是 `Makefile` 的父目录.  
    例如，如果运行 `cd /; make -f /home/username/project/Makefile`, `current_dir` 返回的是 `/` 而不是 `/home/username/project/`

    以下代码适用于任何目录调用的 `Makefile`
    ```
    mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
    current_dir := $(notdir $(patsubst %/,%,$(dir $(mkfile_path))))
    ```

14. 如果没有指定目标，make 如何知道要构建的默认目标  
    默认情况下，`make` 首先处理不以 `.` 开头的第一个目标（又叫「默认目标」）.  
    也可以使用 `.DEFAULT_GOAL` 指定默认目标（`make` <= 3.80 将不起作用）
    ```
    .DEFAULT_GOAL := mytarget
    ```
