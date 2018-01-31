nohup python 脚本，重定向输出，日志内容为空
===========================================

在服务器，开启了一个`守护进程`，执行 `python` 脚本，并且输出重定向到日志记录
```
$ nohup python xxx.py > log &
```

执行一段时间，查看日志
```
$ tail -f log
```
发现， 内容为空.

脚本里有用到 `sys.stdout.write()` 和 `print()` 输出的，但是日志并没有记录到内容.  

怀疑脚本有问题， 直接在命令端执行
```
$ python xxx.py
```
发现脚本正常，有内容输出.

Google了下， 发现是 `Python` `输出缓冲` 的原因.
> `Python` 对标准输出(stdout)默认是有缓冲管理的, 而标准错误输出(stderr)是没有缓存的.
>
> 也就是说， 在程序中尽管 `print` 输出好几行，但是如果还没有占满缓存区的话，只有等待缓存区满或者程序结束之后才会一次性输出.

避免输出缓存的话，可以每次在 `sys.stdout.write()` 和 `print()` 之后调用 `sys.stdout.flush()` 强制刷新缓冲区输出.  
```Python
import sys

print('Hello World')
sys.stdout.flush()

# 或者
sys.stdout.write('Hello World')
sys.stdout.flush()
```

对于 `print()`，还可以
```
>>> print('Hello World.', flush=True)
```
详细查看: [print()](https://docs.python.org/3/library/functions.html#print)

也可以在脚本执行的时候直接使用 `-u` 选项避免缓冲
```
$ nohup python -u xxx.py > log &
```
