ACL 策略，Linux 文件访问控制列表
================================

**访问控制表（Access Control List，ACL）**，又称**存取控制串列**，是使用以存取控制矩阵为基础的存取控制方法，每一个物件对应一个串列主体。  
存取控制串列描述每一个物件各自的存取控制，并记录可对此物件进行存取的所有主体对物件的权限。  
> 摘自 [WiKi](https://www.wikiwand.com/zh-hans/%E5%AD%98%E5%8F%96%E6%8E%A7%E5%88%B6%E4%B8%B2%E5%88%97)

而 **ACL策略** 由一个或多个条目组成，这些条目描述了：  
- 对对象的访问权明确受控的用户和组的名称
- 允许每个用户，组或角色执行特定的操作
- 允许特殊 `any-other` 和 `unauthenticated` 用户类别执行特定的操作

`Linux` 可以基于普通文件或目录设置 `ACL 策略` , 简单来说，就是给指定的用户或组设置某个文件的操作权限.

#### getfacl 
> 查看文件的 `ACL` 规则  
> – getfacl 文档  

#### setfacl 
> 设置文件的 `ACL` 规则  
> – setfacl  -m u:用户名:权限类别 文档  
> – setfacl  -m g:组名:权限类别 文档  
> – setfacl  -x u:用户名 文档 #删除指定的ACL策略  
> – setfacl  -b 文档 #清空ACL策略  

#### 例子演示
```
(cryven) $ su 
(root)   # mkdir demo
(root)   # ls -l 
drwxr-xr-x 2 root   root  4096 Dec 16 14:35 demo

(root)   # chmod -R o=--- demo/
(root)   # ls -l
drwxr-x--- 2 root   root  4096 Dec 16 14:35 demo

(root)   # exit
(cryven) $ cd demo
bash: cd: demo/: Permission denied

(cryven) $ getfacl demo/
# file: demo/
# owner: root
# group: root
user::rwx
group::r-x
other::---

(root)   # setfacl -m u:cryven:rx demo/
(cryven) $ getfacl demo/
# file: demo/
# owner: root
# group: root
user::rwx
user:cryven:r-x
group::r-x
mask::r-x
other::---

(cryven) $ cd demo/
(cryven) $ pwd
/home/cryven/demo

(root)   # setfacl -x u:cryven demo/ # 删除acl规则
(root)   # getfacl
# file: demo/
# owner: root
# group: root
user::rwx
group::r-x
other::---
```
