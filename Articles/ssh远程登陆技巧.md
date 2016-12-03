---
title: ssh远程登陆技巧
date: 2016-08-13 00:05:58
tags: 
- ssh 
categories: Linux
---
在Linux下ssh远程登陆的时候，可以简单设置一下，实现远程登陆免密码，利用别名解析IP。

#### 利用ssh key在网络上畅通无阻 ####
ssh中，有两个钥匙：**公钥，私钥**。   
其中，公钥主要是对一些敏感信息进行加密的，私钥是用于解密的。

ssh的文件都存在~/.ssh中，其中有：
- 客户端：id_ras(私钥匙)，id_rsa.pub(公钥)，known_hosts(已知远程主机)    
- 服务器：authorized_keys(验证过的公钥列表)，sshd_config(ssh配置文件)    

知道了这些文件的用途之后，下面我们简单配置一下，实现Linux无密码登陆。  
1. 客户端使用`ssh-keygen`生成密钥对（私密和公密）
2. 复制公密（id_rsa.pub）的内容
3. 远程登陆服务器端，把复制到的公密内容粘贴到`authorized_keys`  

然后，就可以无密码地`ssh`了，告别手动输入一串随机的密码字符串！

#### 告别ip地址，别名登陆 ####
用ssh登陆的时候，经常要输入类似：  

    $ ssh root@119.119.1.38

对于多个服务器的时候，输入这些IP就有点崩溃了。  
幸好`ssh`自带别名方式，充分地解决了这一问题。  
1. 修改本地`ssh`配置文件
```
$ vim ~/.ssh/config
```
2. 添加别名
```
# serverName
Host serverName
HostName 119.119.1.38
User root
Port 28888
IdentifyFile ~/.ssh/id_rsa
```
3. 配置完成之后，可以直接使用：
```
$ ssh serverName
```
以上就是`ssh`登陆的一些小技巧。配置好之后，就畅游各个服务器吧。:)
