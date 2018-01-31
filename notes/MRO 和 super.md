MRO 和 super
=============

场景
-----
在《流畅的Python》的第八章第七节说到调用父类方法，其中有段示例代码：
```Python
class Base :
    def __init__ (self):
        print ('Base.__init__')
class A (Base):
    def __init__ (self):
        super() . __init__()
        print ('A.__init__')
class B (Base):
    def __init__ (self):
        super() . __init__()
        print ('B.__init__')
class C (A,B):
    def __init__ (self):
        super() . __init__() # Only one call to super() here
        print ('C.__init__')


if __name__ == '__main__':
    c = C()
```

看了代码，以为会输出：
```
Base.__init__
A.__init__
B.__init__
C.__init__
```

但，结果是：
```
Base.__init__
B.__init__
A.__init__
C.__init__
```

发现对 `Python` 的 `MRO` 和 `super()` 掌握的不好，就特意查了资料，做了这篇学习笔记。

MRO
----
`MRO` 全称 `Method Resolution Order`，中文叫「方法解析顺序」.

`MRO` 列表遵循如下三条准则：
- 子类会优先于父类被检查
- 多个父类会根据它们在列表中的顺序被检查
- 如果对下一个类存在两个合法的选择，选择第一个父类

`Python3` 的 `MRO` 是采用 `C3` 算法，详细参考这篇文章：[Python的方法解析顺序(MRO)](http://hanjianwei.com/2013/07/25/python-mro/)  
这里就不做赘述了。

super
-----
`super()` 函数实质上是：
```Python
# inst -> instance
def super(cls, inst):
    mro = inst.__class__.mro() # Always the most derived class
    return mro[mro.index(cls) + 1]
```

> PS: **不要一说到 super 就想到父类！super 指的是 MRO 中的下一个类！**

强烈推荐阅读这篇文章：[理解 Python super](https://laike9m.com/blog/li-jie-python-super,70/)

结论
-----
回到 `场景` 的示例代码，当 `c = C()` 的时候：  
```
In [3]: c.__class__
Out[3]: __main__.C

In [4]: c.__class__.mro()
Out[4]: [__main__.C, __main__.A, __main__.B, __main__.Base, object]
```
可以看到对象 `c` 是 `instance of C`，以及 `C` 的 `MRO` 列表.

`C.__init__()` 中的 `super().__init__()` 实则上是：  
`super(C, self).__init__()`, 而 `super(C, self)` -> `self.__class__.mro(0+1)` -> `__main__.A`  
所以 `super(C, self).__init__()` 相当于 `A.__init__()`  
> 注意：这里的 `self` 是 `instance of C`

而 `A.__init__()` 中的 `super().__init__()` -> `super(A, self).__init__()`, `super(A, self)` -> `self.__class__.mro(1+1)` -> `__main__.B`  
则 `super(A, self).__init__()` -> `B.__init__()`
> 注意：这里的 `self` 是 `instance of C`

以此类推，`c = C()` -> `C.__init__` -> `super` -> `A.__init__` -> `super` -> `B.__init__` -> `super` -> `Base.__init__`

最后输出：
```
Base.__init__
B.__init__
A.__init__
C.__init__
```
