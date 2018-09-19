strtotime 函数
==============

### 函数说明
`strtotime()` — 将任何字符串的日期时间描述解析为 Unix 时间戳

```php
int strtotime ( string $time [, int $now = time() ] )
```
> 注意 $time 的日期与时间格式（见底部参考）

### 局限
`strtotime()` 在某些情况下 `+1 month` 的返回结果异常.

```php
$test = strtotime('2017-12-31');

echo date('Y-m-d', strtotime('+1 month', $test));
# 期待：2018-01-31
# 输出：2018-01-31

echo date('Y-m-d', strtotime('+2 month', $test));
# 期待：2018-02-28
# 输出：2018-03-03
```

正确的用法
```php
$test = strtotime('2017-12-31');

echo date('Y-m-d', strtotime('last day of +1 month', $test));
# 输出：2018-01-31
echo date('Y-m-d', strtotime('last day of +2 month', $test));
# 输出：2018-02-28
```
在使用 `strtotime()` 函数的时候，指定每个月的最后一日，防止 `+1 month` 「溢出」异常结果.

### 不相关例子

#### 场景
一个定时任务，每个月的某天某时某分触发（则当前时间可知），求这个月的最后一天日期.

```bash
$ crontab -l
0 0 20 * * /usr/bin/php -f index.php # 每个月20号 0:00 执行
```

#### 方法一
```php
$nextMonth    = strtotime('+1 month'); // now => 2018-09-19 21:57:59
$nextFirstDay = date('Y-m', $nextMonth) . '-01';
$lastDay      = date('Y-m-d', strtotime('-1 day', strtotime($nextFirstDay)));

echo $lastDay;
# 输出：2018-09-30
```
> 一面试官讲解的方法

#### 方法二
```php
echo date('Y-m-t', time()); // time() => 2018-09-19 21:57:59
# 输出：2018-09-30
```

#### 方法三
```php
echo date('Y-m-d', strtotime('last day of this month')); // now => 2018-09-19 21:57:59
# 输出：2018-09-30

echo date('Y-m-d', strtotime('last day of +0 month')); // now => 2018-09-19 21:57:59
# 输出：2018-09-30
```

**参考：**  
[PHP Manual](http://php.net/manual/zh/function.strtotime.php)  
[日期与时间格式](http://php.net/manual/zh/datetime.formats.php)
