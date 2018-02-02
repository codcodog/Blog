### 服务器配置

#### 创建空仓
`ssh` 到服务器, 创建一个空仓
```Bash
$ mkdir blog.git
$ cd blog.git
$ git init --bare --shared
```
`--bare`：创建一个空仓  
`--shared`：指定仓库可以多个用户共享

#### 创建钩子
```Bash
$ vi hooks/post-receive
```

输入如下内容：
```Bash
#!/bin/bash

GIT_WORK_TREE=/var/www/html/blog git checkout -f
```
> 注意：`/var/www/html/blog` 目录必须存在，否则客户端在提交到服务器仓库的时候，会报 `fatal: This operation must be run in a work tree` 错误

赋予执行权限
```
$ chmod a+x post-receive
```

#### 创建帐号
```Bash
$ useradd -g git -s /usr/local/git/bin/git-shell -m blog
```
> 注意：要为用户创建家目录，否则客户端提交到服务器仓库的时候，会报 `Could not chdir to home directory /home/blog: No such file or directory` 错误

设置帐号密码，在客户端推送内容的时候用到.
```
$ passwd blog
```

另外，对于 `blog.git` 目录，用户要拥有 `可写` 权限才能进行 `推送`.
```Bash
$ chown blog:git -R blog.git
$ chmod -R g+w blog.git
```

### 客户端配置
```Bash
$ cd blog
$ git init .
$ git add .
$ git commit -m 'first commit'
$ git remote add origin blog@YOUR_SERVER_IP:/PATH/TO/blog.git 
$ git push -u origin master
```
> 注意：如果你服务器不是使用默认的 `SSH` 端口 22, 则在添加远程仓库地址的时候使用 `ssh` 协议

```
$ git remote add origin ssh://blog@YOUR_SERVER_IP:PORT/PATH/TO/blog.git
```

参考文章：  
[服务器上的 Git - 在服务器上搭建 Git](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E6%90%AD%E5%BB%BA-Git)
