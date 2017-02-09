修改Mysql数据文件存放目录
=========================

场景：  
随着数据存储越来越多，服务器上的磁盘已经满了，需要把Mysql数据文件存放在另外一个磁盘下。  

1. 登陆数据库查看数据存放的目录
```
$ mysql -uroot -p

mysql> show variables like '%dir%';
+-----------------------------------------+----------------------------+
| Variable_name                           | Value                      |
+-----------------------------------------+----------------------------+
| basedir                                 | /usr/                      |
| binlog_direct_non_transactional_updates | OFF                        |
| character_sets_dir                      | /usr/share/mysql/charsets/ |
| datadir                                 | /var/lib/mysql/            |
| innodb_data_home_dir                    |                            |
| innodb_log_group_home_dir               | ./                         |
| innodb_max_dirty_pages_pct              | 90                         |
| plugin_dir                              | /usr/lib64/mysql/plugin    |
| slave_load_tmpdir                       | /tmp                       |
| tmpdir                                  | /tmp                       |
+-----------------------------------------+----------------------------+
```
其中`datadir`存储的就是数据文件。  

2. 停止Mysql服务进程
```
$ service mysqld stop
```
或者：  
```
$ /etc/init.d/mysqld stop
```

3. 创建数据目录并把数据文件复制过来
```
$ mkdir /home/data/mysql
$ cp -R /var/lib/mysql/* /home/data/mysql
```

4. 修改目录权限
```
$ ls -l /var/lib/ | grep mysql # 查看之前mysql目录的权限
$ chown mysql:mysql -R /home/data/mysql
```

5. 修改my.cnf
```
$ vim /etc/my.cnf
```
把`datadir`修改为`/home/data/mysql`

6. 修改启动脚本
```
$ vim /etc/init.d/mysqld
```
把`datadir`修改为`/home/data/mysql`

7. 启动Mysql服务
```
$ service mysqld start # 或者 /etc/init.d/mysqld start
```

最后，重复1操作，查看数据存放目录是否更改成功。  

