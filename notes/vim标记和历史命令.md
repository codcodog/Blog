Vim小技巧
=========

标记
----
vim允许在文本中自定义标记。  
> 命令`ma`指用a标记了当前的光标位置.  

移动到标记a的命令是：  
> \`a，(反引号，跟～共用一个键盘)  

需要注意的是，单引号`'a`也可以实现跳转，不同的是：  
> 单引号跳转到标记所在行的行首  
>
> 而反引号是直接跳到标记所在的行和列，精确到列的。  
> 因此，在同一行多列之间跳转使用反引号是非常有用的。  

历史命令
--------
使用vim的都知道，`:q`是退出vim。  
而很少人知道，`q:`是查看vim里使用的历史命令。  
> 最有趣的是，你还可以在这个历史命令窗口修改命令，按enter就执行此命令了。  
