---
title: 搭建ＣＩ环境遇到的问题
date: 2016-07-19 21:50:32
tags: 
- CI生产环境
- 基础
categories: phper
---
今天，终于解决了困扰了我好几天的问题：**搭建ＣＩ生产环境，正确的路由（ＵＲＬ），却无法找到页面。**
之后，在搭建微信订阅号的那个环境时，也是遇到相同问题。　　
访问的路径正确，却无法找到页面。

首先，要说明下，之前这两个项目都是在ｗｉｎｄｏｗｓ环境下的，相同的路由（ＵＲＬ）在ｗｉｎｄｏｗｓ环境下可以正确访问，而在Ｌｉｎｕｘ，也就是服务器上却报错，页面找不到。（好明显，原因８０％是大小写问题，可惜之前我一直没探究这个问题，还以为是自己服务器配置问题，囧......）

现在，就还原一下问题确切原因的情景。
在这里，我就不以ＣＩ框架为例子了，原因应该是跟我自己做的那个微信搭建原因一致。下面，我就以自己微信做的例子来说明一下。　　

自己做的那个微信订阅号，是自己用了一些很基本的框架原理来进行开发的。其中，里面用到了这样的一个魔术方法：__autoload()。看代码：

```
function __autoload($class){
        if(strtolower(substr($class,-5)) == 'model'){
                include(ROOT.'model/'.$class.'.class.php');
        }elseif (strtolower(substr($class,-3) == 'tpl')){
                include(ROOT.'class/'.strtolower($class).'.class.php');
        }else {
                include (ROOT.'common/'.$class.'.class.php');
        }
}
```
而我自己定义的类名，一般都是用小写字母（当时在ｗｉｎｄｏｗｓ系统开发，对大小写不敏感）。例如，我用到的几个ｃｌａｓｓ，都是这样命名的：`basic_tpl.class.php`,　`url_tpl.class.php`,　`xiaoji_tpl.class.php`等。
这样是不符合规范的，正确的应该是首字母大写的，`Basic_tpl.class.php`。
因为，我们的类在声明时候，是这样的：
```
class Basic_tpl{
	protected $postObj;  //simplexml对象
	protected $fromUser; //微信用户
	protected $toUser;   //微信公众号
	protected $time;     //时间戳
	protected $MsgType;  //消息类型 
	protected $content;  //消息内容
	protected $result;   //重组之后的数据
	
	public $response;  //回复消息内容
	
	public function __construct($postStr){
		$this->getObj($postStr);
		.......
```
而在 `__autoload($class)` 这个魔术方法中，自动传过来的  `$class = 'Basic_tpl'`，那么，在 `include`的时候，好明显在Ｌｉｎｕｘ是 `include` 了这个文件 `Basic_tpl.class.php`，而不是 `basic_tpl.class.php`。如果实在ｗｉｎｄｏｗｓ系统下，程序是可以正确运行的，但是在Ｌｉｎｕｘ（对大小写敏感），就会出现困扰了我好几天的**页面找不到！**。　　

所以，我把所有的类文件都修改了首字母大写。然后，问题就解决了。
对于，ＣＩ框架的也是一样，我把自己的类文件首字母改为大写，问题也解决了。

** 总结：不管在什么系统下开发，都要遵循规范的格式进行编程！**（最好还是用Ｌｉｎｕｘ吧！你会发现它的美！）



