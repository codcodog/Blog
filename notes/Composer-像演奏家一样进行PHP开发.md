---
title: 'Composer,像演奏家一样进行PHP开发'
date: 2016-08-13 11:25:10
tags: 
- php
- dependency management
categories: phper
---
#### Composer ####
对于现代语言而言，包管理器基本上是标配。Java 有 Maven，Python 有 pip，Ruby 有 gem，Nodejs 有 npm。PHP 的则是 PEAR，不过 PEAR 坑不少:
> 依赖处理容易出问题  
> 配置非常复杂  
> 难用的命令行接口  

好在我们有**Composer**，PHP依赖管理的利器。它是开源的，使用起来也很简单，提交自己的包也很容易。  
> 简单来说，Composer 是一个新的安装包管理工具，服务于 PHP 生态系统。它实际上包含了两个部分：Composer 和 Packagist。  

**[Composer](http://www.phpcomposer.com/)** 是由 Jordi Boggiano 和 Nils Aderman 创造的一个命令行工具，它的使命就是帮你为项目自动安装所依赖的开发包。Composer 中的很多理念都借鉴自 npm 和 Bundler，如果你对这两个工具有所了解的话，就会在 composer 中发现他们的身影。Composer 包含了一个依赖解析器，用来处理开发包之间复杂的依赖关系；另外，它还包含了下载器、安装器等有趣的东西.

**[Packagist](https://packagist.org/)** 是 Composer 的默认的开发包仓库。你可以将自己的安装包提交到 packagist，将来你在自己的 VCS （源码管理软件，比如 Github）仓库中新建了 tag 或更新了代码，packagist 都会自动构建一个新的开发包。这就是 packagist 目前的运作方式，将来 packagist 将允许直接上传开发包。

##### Composer's benefits #####
> 1. Quickly integrate libraries for your SaaS providers (pusher, algolia, aws, opentok, twilio, stripe, and many others). Get the package name and version, add them to your composer.json and run the install command.  
> 2. Ability to use ready made packages that solve common problems. You need a **routing** package? search for **routing** on packagist and get started right away. You need to handle **uploaded** files? Search for **upload** on packagist and get started right away.  
> 3. Autoload all your classes using Composer’s autoload https://getcomposer.org/doc/04-schema.md#autoload. Useful for getting rid of requires in your code.  
> 4. Customize your composer workflow with Composer scripts. You can run your own scripts before/after composer install, before/after composer update, etc.

#### Composer的安装 ####
```
$ curl -Ss https://getcomposer.org/installer | php
```

(博主是Linux系统)直接在命令行执行上面的命令，会在本地得到`composer.phar`。然后，  
```
$ mv composer.phar /usr/local/bin/composer  
```

另外要说明的是，在系统`PATH`中需要`export`一下`/usr/local/bin`这个路径，要不没法在命令行全局调用。  
完成上面的操作之后，我们就可以开始使用`composer`了。  
```
$ composer
```
如果是Windows系统，可以参考：
> Windows installer: Visit https://getcomposer.org/download/ and download the Composer-Setup.exe

#### Composer的使用 ####

##### Installing the package #####
安装包有两种方式，例如：需要安装`packagist`上的[monolog/monolog](https://packagist.org/search/?q=monolog%2Fmonolog)包。  
在`terminal`输入：  
```
$ composer require monolog/monolog   
```
这样子，就完成了一次包的安装。
另外一种方式是配置`composer.json`文件来完成安装的。  
```
    {
        "require": {
            "monolog/monolog": "1.12.0"
        }
    }
```
配置`composer.json`之后，回到`terminal`,
```
$ composer install
```

完成安装。  

另外需要说明的是，`$ composer install --no-dev`中`--no-dev`是指不要下载开发版的，跟下面的类似：
> **DEV-MASTER**  
> By specifying `dev-master` you are grabbing the currently developed latest release which hasn’t been tagged with a version number yet. This can be just fine while developing but you need to be aware that the potential for bugs is higher in these versions.

##### How to add packages #####
这跟安装包一样，可以直接在`terminal`安装：
```
$ composer require fzaninotto/faker
```

或者，配置`composer.json`文件，然后`$ composer install`.最后，`composer.json`内容如下：
```
{
    "require": {
        "monolog/monolog": "~1.13",
        "fzaninotto/faker": "1.4.0"
    }
}
```

##### How to update the packages #####
当我们想更新下载包的时候:
```
$ composer update
```
> As a result of this command, Composer will automatically download all the updated versions of the packages, including all the dependencies, straight to the project folder.

##### How to remove a package from Composer #####
In order to remove one of the packages, we need to delete the line corresponding to the package from the `composer.json` file and update compser
```
$ composer update
```

This will remove the package with all of it dependencies from the vendor directory.

最后，对于`Composer`的简单学习就此结束了。对于一些其他用法或者一些高级特性可以参考`Composer`官网：https://getcomposer.org/  


#### 如何使用第三方类库 ####
对于下载回来的包，只要把`vendor`目录下`autoload.php`文件引入即可。例如：
```
<?php
require_once __DIR__ . '/vendor/autoload.php';

// use the factory to create a Faker\Generator instance
$faker = Faker\Factory::create();

// generate data by accessing properties
echo $faker->name;

// generate fake address
echo $faker->address;

// generate fake name
echo $faker->text;
```

参考文章：  
[How to use Packagist and Composer to integrate existing code libraries into your PHP apps?](http://phpenthusiast.com/blog/how-to-use-packagist-and-composer)  
[A Beginner’s Guide To Composer](https://scotch.io/tutorials/a-beginners-guide-to-composer)  
[PHP Tutorial: Getting Started with Composer](https://www.codementor.io/php/tutorial/composer-install-php-dependency-manager)  
[Compser中文网](http://www.phpcomposer.com/)

