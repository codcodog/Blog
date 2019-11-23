coding 配置 ssh 公匙无效
=========================

### 场景

在 coding 后台添加了「SSH 公钥」，但是无效  
报 `client_loop: send disconnect: Broken pipe` 错误.
```bash
~ $ ssh -T git@git.coding.net
client_loop: send disconnect: Broken pipe
```


### 原因

可能是虚拟机 `VMWare` 的原因（不确定）.
> 在自己的笔记本配置，没有出现以上问题，但在虚拟机配置则有.

在 `~/.ssh/config` 添加以下内容
```
Host *
    ServerAliveInterval 600
    TCPKeepAlive yes
    IPQoS=throughput
```
「SSH 公钥」就生效了.

### 参考
[SSH 公钥使用办法](https://dev.tencent.com/help/doc/faq/bbe781aee786/ssh)  
[VMWare SSH Bug](https://blog.bchoy.me/post/2018-09-11-vmware-ssh-bug/)  
