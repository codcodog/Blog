Cron 语法备份
=============

cron常用命令
------------
`crontab  -e` 使用默认编辑器(vi/emacs)编辑 cron tables.  
`crontab -l` 查看计划任务.  
`crontab -r` 删除任务

cron table syntax
------------------
```
________________________ Minute of the Hour (0-59)
| _____________________ Hour of the Day (0-23)
| | __________________ Day of the Month (1-31)
| | | _______________ Month of the Year (1-12)
| | | | _________ Day of the Week (0 - 6 for Sunday through Saturday)
| | | | | _________ Command to Execute (Full path is required)
| | | | | |
| | | | | |
1 3 8 1 * /usr/local/bin/somescript.Bash 2>&1 >/dev/null
```

