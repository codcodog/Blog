mysql 8.0 重置 root 密码
=========================

### 场景
`mysql 8.0` 安装之后，忘记了 `root` 密码，需要重置.

### 方案

编辑 `my.cnf`
```
$ vi /etc/my.cnf
```

在 `[mysqld]` 下添加 `skip-grant-tables`
```
[mysqld]
skip-grant-tables
```

重启 `mysql`
```
$ systemctl restart mysqld.service
```

重新登录 `mysql`，无需密码，直接 `enter` 进入
```
$ mysql -uroot -p
```

进入 `mysql` 之后，要先 **刷新权限**，否则无法修改密码
```
mysql> flush privileges;
```

修改密码
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```
> mysql 8.0 默认是安装了 `validate_password` 插件的  
> 密码至少包含一个大写字母，一个小写字母，一位数字和一个特殊字符，并且密码总长度至少为8个字符

设置完成之后，编辑 `my.cnf` 去除 `skip-grant-tables` 指令.

之后重启 `mysql` 即可
```
$ systemctl restart mysqld.service
```

### 更多

#### 设置 root 远程登录

登录 `mysql` 之后，查看权限
```
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select host, user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | root             |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
+-----------+------------------+
4 rows in set (0.00 sec)
```

修改权限，允许远程访问
```
mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

#### yum 安装 mysql

参考官方的 [A Quick Guide to Using the MySQL Yum Repository](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)
