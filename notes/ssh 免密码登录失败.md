ssh 免密码登录失败
==================

### 场景
本地有一个 `username1@Server:/path/to/repo.git` 的仓库，每次 `push` 代码的时候都要输入密码，想设置为免密 `push`.

由于仓库使用的是 `ssh`, 因此免密 `push` 其实就是：`ssh` 免密码登录.  

### ssh 免密码登录
客户端使用 `ssh-keygen` 生成公钥和私钥
```
$ ssh-keygen -t rsa
```
输入后，生成两个文件： `~/.ssh/id_rsa` (私钥), `~/.ssh/id_rsa.pub` (公钥)

将公钥的内容复制到服务器端的 `authorized_keys`
```
$ ssh-copy-id username1@Server
```

或者
```
$ scp ~/.ssh/id_rsa.pub username1@Server:/.ssh/authorized_keys  # 确保服务器端 ~/.ssh 目录存在
```

### 遇到的问题
#### 问题一
由于用户 `username1` 在服务器端的 `shell` 为 `git-shell`，在使用 `ssh-copy-id` 的时候报错：  
```
fatal: unrecognized command '
        umask 077 ;
        mkdir -p .ssh && cat >> .ssh/authorized_keys || exit 1 ;
        if type restorecon >/dev/null 2>&1 ; then restorecon -F .ssh .ssh/authorized_keys ; fi'
```
`git-shell` 的原理可以参考这篇文章：[GIT-SHELL 沙盒绕过](https://www.leavesongs.com/PENETRATION/git-shell-cve-2017-8386.html)

#### 问题二
因此，直接采用第二种方法，使用 `scp` 把公钥 `~/.ssh/id_rsa.pub` 复制到服务器端.

由于 `git-shell` 用户不能登录服务器，我们这里使用 `root` 用户
```
$ scp ~/.ssh/id_rsa.pub root@Server:/home/username1/.ssh/authorized_keys
$ ssh root@Server

# cd /home/username1
# chmod -R 0777 .ssh
# exit
```
完成配置之后，发现 `push` 代码的时候，还是要输入密码，配置不起作用.

### 解决的办法
对于 `scp` 的问题，发现是权限设置的原因.

要确保 `.ssh` 的权限为：  
属主 `username1`, 属组 `git`, 并且权限赋予为 `0700`  

`.ssh/authorized_keys` 的权限为：  
属主 `username1`, 属组 `git`, 并且权限赋予为 `0600`  

> 关键点：确保 username1 家目录内容不被其他用户修改

```
$ ssh root@Server

# cd /home/username1
# chown username1:git -R .ssh
# chmod 0700 .ssh
# chmod 0600 .ssh/authorized_keys
```
最后，`push` 代码，就不用再输入密码了.

#### 为什么
查看 `ssh` 手册
```
$ man ssh 
 ~/.ssh/
         This directory is the default location for all user-specific configuration and authentication information.  There is no general requirement to keep the entire contents of this directory secret,
         but the recommended permissions are read/write/execute for the user, and not accessible by others.

 ~/.ssh/authorized_keys
         Lists the public keys (DSA, ECDSA, Ed25519, RSA) that can be used for logging in as this user.  The format of this file is described in the sshd(8) manual page.  This file is not highly sensi‐
         tive, but the recommended permissions are read/write for the user, and not accessible by others.

```

查看 `sshd` 手册
```
$ man 8 sshd
~/.ssh/authorized_keys
         Lists the public keys (DSA, ECDSA, Ed25519, RSA) that can be used for logging in as this user.  The format of this file is described above.  The content of the file is not highly sensitive, but
         the recommended permissions are read/write for the user, and not accessible by others.

         If this file, the ~/.ssh directory, or the user's home directory are writable by other users, then the file could be modified or replaced by unauthorized users.  In this case, sshd will not
         allow it to be used unless the StrictModes option has been set to “no”.
```
可以看到，当 `authorized_keys` 可以被其他用户修改的时候，`sshd` 将不允许使用它，除非 `StrictModes` 设置为 `no`.
