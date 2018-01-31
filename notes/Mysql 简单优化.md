Mysql 简单优化
==============

#### 场景
使用 `Python` 连接 `Mysql` 查询数据库相关信息，由于`高并发`：查询速度快，而且采用线程并发，发现服务器上 `Mysql` 程序占用 `CPU` 过高.

以下是部分代码：
```Python
t2   = time.time()
res  = self.db.get_code_data(symbol)
t3   = time.time()
t4   = t3 - t2
```

```Python
def get_code_data(self, code):
''' 获取code数据
'''
sql = "select `date`, `value` from `close` where `code`={} and `code_type` = 'hk' order by `date` DESC".format(code)
res = self.multi_query(sql)

return res
```

#### 优化
上 `Mysql` 服务器， 登陆 `Mysql`
```
MariaDB [(none)]> SHOW PROCESSLIST;
+--------+------+----------------------+------------+---------+------+--------------+--------------------------------------------------------------------------------------------------+----------+
| Id     | User | Host                 | db         | Command | Time | State        | Info                                                                                             | Progress |
+--------+------+----------------------+------------+---------+------+--------------+--------------------------------------------------------------------------------------------------+----------+
| 147322 | root | 119.137.52.xxx:24672 | xxxxsao    | Sleep   | 7663 |              | NULL                                                                                             |    0.000 |
| 147363 | root | 119.137.52.xxx:22012 | NULL       | Sleep   | 7162 |              | NULL                                                                                             |    0.000 |
| 147394 | root | 119.137.52.xxx:22439 | xxxxx_test | Sleep   | 5797 |              | NULL                                                                                             |    0.000 |
| 147395 | root | 119.137.52.xxx:22454 | xxxxx_test | Sleep   | 6026 |              | NULL                                                                                             |    0.000 |
| 147396 | root | 119.137.52.xxx:22521 | xxxxx      | Sleep   | 6009 |              | NULL                                                                                             |    0.000 |
| 147397 | root | 119.137.52.xxx:22575 | xxxxx      | Sleep   | 6009 |              | NULL                                                                                             |    0.000 |
| 147400 | root | 119.137.52.xxx:23336 | NULL       | Sleep   | 5799 |              | NULL                                                                                             |    0.000 |
| 147428 | root | 119.137.52.xxx:24422 | xxxxx_test | Sleep   | 5337 |              | NULL                                                                                             |    0.000 |
| 147494 | root | 119.137.52.xxx:21561 | xxxxsao    | Sleep   | 4735 |              | NULL                                                                                             |    0.000 |
| 147774 | root | 119.137.52.xxx:23651 | NULL       | Sleep   | 4106 |              | NULL                                                                                             |    0.000 |
| 147775 | root | 119.137.52.xxx:23652 | NULL       | Sleep   | 4106 |              | NULL                                                                                             |    0.000 |
| 147809 | root | 119.137.52.xxx:22161 | xxxxx_test | Sleep   | 3516 |              | NULL                                                                                             |    0.000 |
| 147916 | root | 119.137.52.xxx:22830 | NULL       | Sleep   | 1066 |              | NULL                                                                                             |    0.000 |
| 147917 | root | 119.137.52.xxx:22875 | xxxxsao    | Sleep   |  951 |              | NULL                                                                                             |    0.000 |
| 147918 | root | 119.137.52.xxx:22930 | NULL       | Sleep   | 1066 |              | NULL                                                                                             |    0.000 |
| 147926 | root | 119.137.52.xxx:23288 | xxxxsao    | Sleep   |  944 |              | NULL                                                                                             |    0.000 |
| 147938 | root | 119.137.52.xxx:23334 | xxxxsao    | Sleep   |    3 |              | NULL                                                                                             |    0.000 |
| 147950 | root | 119.137.52.xxx:24339 | xxxxsao    | Sleep   |    3 |              | NULL                                                                                             |    0.000 |
| 147956 | root | 119.137.52.xxx:24599 | xxxxsao    | Sleep   |    2 |              | NULL                                                                                             |    0.000 |
| 147958 | root | localhost            | NULL       | Query   |    0 | NULL         | SHOW PROCESSLIST                                                                                 |    0.000 |
| 147964 | root | 119.137.52.xxx:21972 | xxxxsao    | Query   |    1 | Sorting      | select `date`, `value` from `close` where `code`=3389 and `code_type` = 'hk' order by `date` DESC|    0.000 |
| 147966 | root | 119.137.52.xxx:21998 | xxxxsao    | Query   |    1 | Sorting      | select `date`, `value` from `close` where `code`=3383 and `code_type` = 'hk' order by `date` DESC|    0.000 |
+--------+------+----------------------+------------+---------+------+--------------+--------------------------------------------------------------------------------------------------+----------+
```
(PS: 排版放上来貌似乱了，将就着看吧)

这里 `State` 有两个 `Sorting`，这是由于 `SQL` 语句中使用了 `order by`，`Mysql` 每次 `Query` 完之后，还要排好序，才发回给客户端.  
而这个排序，在`高并发`的时候，会使 `Mysql` 服务器的 `CPU` 飙到很高.

简单优化下，把排序操作放在 `Python`，而非 `Mysql`， `Mysql` 把查询到信息直接发回客户端，不做任何处理.
> 也可以选择在 `order by` 的字段添加索引
```Python
t2   = time.time()
res  = self.db.get_code_data(symbol)
t3   = time.time()
t4   = t3 - t2

rows   = res.rowcount

# 如果没有数据，跳过
if rows == 0:
    continue

res1 = sorted(res, key=itemgetter(0), reverse=True)
```
> `res1` 做了关于 `日期` 的排序，从新到旧.

```Python
def get_code_data(self, code):
''' 获取code数据
'''
sql = "select `date`, `value` from `close` where `code`={} and `code_type` = 'hk'".format(code)
res = self.multi_query(sql)

return res
```

这样简单优化之后，`Mysql` 服务器 `CPU` 没有一直久居不下了，但由于查询速度快，并发10个线程，`CPU`占用偶尔还是会到达一个较高的阀值，不过相对之前来说，好了不少.

#### 总结
在 `高并发` 的场景时，`SQL` 尽量减少 `order by` 语句的使用，可以把排序操作放在客户端进行.  
或者，在 `order by` 后面的字段添加 `索引`

(PS: 最后测试发现，添加索引的方法更快捷)
