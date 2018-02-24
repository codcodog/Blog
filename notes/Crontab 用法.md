Crontab 用法
============

### crontab 命令
列出 `cron` 任务
```
$ crontab -l
$ crontab -u USERNAME -l
```

清除所有 `cron` 任务
```
$ crontab -r
```
清除某个用户的 `cron` 任务
```
$ crontab -r -u USERNAME
```

创建 `cron` 任务
```
$ crontab -e
```

`cron` 服务(守护进程)在后台运行，并且不断地检查以下的文件和目录
```
$ /etc/crontab
$ /etc/cron.*/
$ /var/spool/cron/
```

> PS：`cron` 将检查所有 `crontabs` 的修改时间并重新载入已更改的那些时间，因此，无论何时修改 `crontab` 文件，都不需要重新启动 `cron`.

详细参考：`man crontab`

### 语法
```
1 2 3 4 5 /Path/To/Command arg1 arg2
```
- 1: Minute (0-59)
- 2: Hours (0-23)
- 3: Day of month (1-31)
- 4: Month (1-12, or use names)
- 5: Day of week (0-7, 0 or 7 is Sunday, or use names)

详细参考：``man 5 crontab ``

#### 特殊字符
- 星号 (``*``)：为字段指定所有可能的值. 例如：在小时字段，则表示每小时.
- 逗号 (``,``)：指定一个值列表，例如：1, 5, 10, 15
- 破折号 (``-``)：指定一个值的范围，例如：5-10, 等同于 5, 6, 7, 8, 9, 10
- 分隔符 (``/``)： 指定一个步长值，例如：``* / 2`` 在小时字段表示每隔2个小时执行一次

#### 缺陷
从上面的语法可以看出：`crontab` 无法精确到秒级.

当时间点触发的时候 `crontab` 在0～1秒内去执行指定的命令, 因此，我们可以配合 `sleep` 命令来进行秒级的一些简单使用.

例如，每分钟第30秒执行一个命令
```
* * * * * sleep 30; /Path/To/Command arg1 arg2
```

### 例子
每天 00:05 执行某个命令
```
5 0 * * * /Path/To/Command
```

每个月的第一天 14:15 执行脚本
```
15 14 1 * * /path/to/script.sh
```

在工作日 22:00 运行
```
0 22 * * 1-5 /scripts/phpscript.php
```

凌晨0点，2点，4点...23分执行
```
23 0-23/2 * * * /root/scripts/perl/perlscript.pl
```

每周日，04:05 执行
```
5 4 * * sun /path/to/unixcommand
```

**参考文章**  
[How To Add Jobs To cron Under Linux or UNIX](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/)
