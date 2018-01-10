mysql主从复制
=============

主从复制原理
------------

`Master`：记录数据更改操作  
> 启用binlog记录模式（二进制日志）  
> 允许Slave读取binlog日志  

`Slave`：运行2个同步线程  
> Slave_IO：负责连接Master，复制其binlog日志文件到本机的relay-log（中继）文件  
> Slave_SQL：执行本机relay-log文件里的SQL语句，重现Master的数据操作  

MySQL使用3个线程来执行复制功能，其中1个在主服务器上，另两个在从服务器上。  

当发出`START SLAVE`时，从服务器创建一个I/O线程，以连接主服务并让主服务器发送二进制日志。主服务器创建一个线程将二进制日志中的内容发送到从服务器。  

从服务器I/O线程读取主服务器Binlog Dump线程发送的内容并将该数据拷贝到从服务器数据目录中的本地文件中，即中继日志。  

第三个线程是SQL线程，从服务器使用此线程读取中继日志并执行日志中包含的更新。  

`SHOW PROCESSLIST`语句可以查询在主服务器上和从服务器上发生的关于复制的信息。  

主从复制配置
------------
主从复制主要分为两种情况：
1. 主服务器还没有用户数据
2. 主服务器已经存在用户数据

> 两者的主要区别是，当主服务器有用户数据的时候，需要对主服务器进行锁表操作。(如果主服务器没有用户数据，那么文章下面关于锁表的相关操作就不用进行)

现在有两部服务器，`192.168.0.1`主服务器(已有用户数据)，`192.168.0.2`从服务器.  

#### 主库锁表
```
mysql> flush tables with read lock;  

mysql> show master status;   
+-------------------+----------+--------------+------------------+-------------------+    
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |    
+-------------------+----------+--------------+------------------+-------------------+    
| mysql-bin.000001 |      353 |              |                  |                   |    
+-------------------+----------+--------------+------------------+-------------------+    
```
> PS：在这里需要记住file和position的值，后面配置从库的时候需要用到.  

#### 主库备份
```
# mysqldump -uroot -p -B mydb > mydb.sql
```
> 说明：-B参数有建库语句。

#### 主库开表锁
```
mysql> unlock tables;
```

#### 从库导入主库的sql文件
```
# mysql -uroot -p < mydb.sql
```
> 说明：如果mydb.sql文件没有创建库语句，则需要登陆mysql，用source命令导入。  

#### 配置主从库（/etc/my.cnf）
```
# master配置
[mysqld]
server-id       = 1                          # 设置服务器id唯一
log-bin = mysql-bin                          # 启用二进制日志文件
binlog-ignore-db = mysql,information_schema  # 忽略写入binlog的数据库

# slave配置
[mysqld]
server-id = 2
log-bin = mysql-bin
replicate-do-db = kuaiing                    # 只同步kuaiing数据库 
slave-skip-errors = all                      # 忽略因复制出现的所有错误
```
配置完成之后，主从服务器都要重启数据库。  
```
# systemctl restart mariadb.service
```

#### 在主服务器上建立账户并授权slave
```
mysql> insert into mysql.user(host,user,password) values("localhost","user1",password("password1"));
mysql> flush privileges;
mysql> grant replication slave on *.* to user1@192.168.0.2 identified by 'password1'; 
```

#### 配置从库
```
mysql> change master to
-> master_host='192.168.0.1',
-> master_user='user1',
-> master_password='password1',
-> master_log_file='mysql-bin.000001', # 在查看主库状态时的file
-> master_log_pos=353; # 在查看主库状态时的position
```

#### 启动slave同步进程并查看状态
```
mysql> start slave;

mysql> show slave status \G

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.47
                  Master_User: slave1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 57491
               Relay_Log_File: localhost-relay-bin.000003
                Relay_Log_Pos: 57775
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 57491
              Relay_Log_Space: 62542
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.01 sec)
```
> 其中Slave_IO_Running 与 Slave_SQL_Running 的值都必须为YES，才表明状态正常。

#### 验证主从是否同步
在主库插入或删除一条数据，看从库是否发生相应的变化。具体实例，在这里就不列举了。  
