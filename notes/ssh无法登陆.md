ssh无法登陆
===========

用ssh远程登陆服务器的时候，出现这个报错：  
> ssh_exchange_identification: read: Connection reset by peer

但是，其他人登陆却可以，只有用公司网络的登不上去。  

于是，登陆了另外一个台服务器A，中转下，在A服务器ssh服务器B(在公司网络登不上去)，如其愿登陆上去了。  

然后，在服务器B上排查下。查看日志`/var/log/auth.log`发现，没有这个文件。也就没法确切定位登陆失败的原因了。  
又看了下`/var/log/secure`，没有发现什么有用的信息。  

Google了下，据说是ip被禁止的原因。然后去查看`/etc/hosts.allow`和`/etc/hosts.deny`，发现`hosts.allow`里没有信息，于是把自己公司ip(外网ip)加进去了。  
```
sshd:23.105.194.*:allow
```
`service sshd restart` 重启，然而发现并没有作用，还是登陆不上去。  
(PS: 当`hosts.deny`和`hosts.allow`同时设置ip的时候，以`hosts.allow`为准)  

然后查看`hosts.deny`发现里面有很多ip被禁止的。`grep`了一下，没有找到自己公司的ip，好奇怪。  
这边同事又催着要登陆，然后我把`hosts.deny`文件清空，发现可以登陆了。  
```
# cat /dev/null > /etc/hosts.deny
```

具体原因没排查出来，一次失败的排查记录。Org......
