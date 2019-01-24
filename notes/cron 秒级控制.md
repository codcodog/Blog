cron 秒级控制
=============

### 场景
使用 `crontab` 每隔5秒执行一个脚本



### 纠正
`crontab` 最小调度单位是「分钟」，没法控制到「秒」.

```
*/5 * * * * /path/to/script
```
这里的 `/` 是指「步长」，意思是每隔5分钟执行一次脚本.  

`crontab` 详细用法参考：[Crontab 用法](https://github.com/codcodog/Blog/issues/92)



### 方案

#### 方案一
编写脚本
```sh
$ cat cron_seconds.sh
#!/usr/bin/env bash

while true; do
  SCRIPT # 脚本的执行路径
  sleep 5
done
```
`cron` 定时任务设置
```
* * * * * /path/to/cron_seconds.sh
```

#### 方案二
```
$ crontab -l
* * * * * /path/to/script
* * * * * sleep 5;/path/to/script
* * * * * sleep 10;/path/to/script
* * * * * sleep 15;/path/to/script
* * * * * sleep 20;/path/to/script
* * * * * sleep 25;/path/to/script
* * * * * sleep 30;/path/to/script
* * * * * sleep 35;/path/to/script
* * * * * sleep 40;/path/to/script
* * * * * sleep 45;/path/to/script
* * * * * sleep 50;/path/to/script
* * * * * sleep 55;/path/to/script
```
