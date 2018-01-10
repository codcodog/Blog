Git回滚远程仓库
===============

场景
-----
在本地修改了代码，并提交到了远程分支上。  
发现是一个错误的提交，然后本地回滚了历史版本，然而在回滚远程分支时，出现了错误。  

回滚
----
下面是回滚的代码：
```
$ git reflog # 查看版本id(COMMENT_ID)
$ git reset --hard COMMENT_ID
$ git push -f origin master
```

结果出现如下报错：
```
remote: error: denying non-fast-forward refs/heads/master (you should pull first)
To git@dz:/data/dazong.git
 ! [remote rejected] master -> master (non-fast-forward)
 error: failed to push some refs to 'git@dz:/data/dazong.git'
```

Google了下，发现是远程仓库只允许fast forward(强制推送不受影响)，但如果`denyNonFastForwards = true`，则无法强制推送了，也就是`git push -f origin master`无法让远程分支回滚历史版本了。  

所以，设置`denyNonFastForwards = false`之后，`git push -f origin master`就可以回滚远程仓库了。

另外需要特别注意，这个远程回滚操作是非常危险的。因为这个强制推送会覆盖远程仓库的代码，也会导致之前的提交被抹掉。

参考文章：  
[使用强制推送，Luca——Jenkins的开发者抹掉了Github上1个月的提交](http://www.infoq.com/cn/news/2013/12/use-the-force)  
[How to force push a reset to remote repository?](http://stackoverflow.com/questions/10544139/how-to-force-push-a-reset-to-remote-repository)  
[远程仓库版本回退方法](http://blog.csdn.net/fuchaosz/article/details/52170105)  
