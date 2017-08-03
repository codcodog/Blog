Flask 学习
==========

目录
----
- [项目结构](#项目结构)
    + [项目的目录结构是如何定义的](#大型程序的项目结构)
    + [是否具备入口文件](#web框架一般都会有个统一的入口文件)
- [定义路由](#定义路由)
    + 如何定义一个URL
    + 从一个URL映射到(某类)某函数(方法)的整个过程是如何实现的
- [Request & Response](#Request--Response)
    + 获取Request参数(GET, POST)
    + 如何定义Url拦截函数, 即在执行这个Url有前后操作(before_url, after_url).
    + 表单验证(数据过滤)
    + 获取/修改/存储Cookie和Session
    + 请求获取第三方API数据, 即通过Http Get/Post 获取远程数据
    + 修改/输出 Http Header数据
    + 解释/输出 Json数据
    + 如何处理状态码: 404 和 50X
- [模板系统](#模板系统)
    + 如何组织/访问模板文件的目录结构
    + 如何在模板中嵌入代码
    + 模板是否支持继承结构
    + 模板之间如何include
    + 如何自定义模板函数
- [数据库操作](#数据库操作)
    + 是否支持ORM
    + 如何直接使用sql语句访问数据库(而不用orm)
    + 如何定义/组织/初始化 数据表
    + 掌握orm常用操作, 例如: add/delete/字段查询/counts/order by等
- [非框架必备技能(Helpers)](#非框架必备技能(Helpers))
    + 文件上传
    + Log日志
    + 发送Email
    + 图片处理

项目结构
--------

### 大型程序的项目结构
```
|-flasky
 |-app/
  |-templates/
  |-static/
  |-main/
   |-__init__.py
   |-errors.py
   |-forms.py
   |-views.py
  |-__init__.py
  |-email.py
  |-models.py
 |-tests/
  |-__init__.py
  |-test*.py
 |-venv/
 |-requirements.txt
 |-config.py
 |-manage.py
```
说明:  
- Flask程序一般都保存在`app`包中
- 单元测试编写在`tests`包中
- `venv`文件夹包含Python虚拟环境

- `templates`存储视图文件(html)
- `static`存储静态文件(css, js等)
- `main`存储Flask程序

- `requirements.txt`列出了所有的依赖包, 便于在其他电脑中重新生成相同的虚拟环境.
    > `pip`可以使用如下命令自动生成这个文件:  

    ```
    (venv) $ pip freeze > requirements.txt
    ```
    > 如果你要创建这个虚拟环境的完全副本, 可以创建一个新的虚拟环境, 并在其上运行以下命令:  

    ```
    (venv) $ pip install -r requirements.txt
    ```
- `config.py`存储配置设置.
- `manage.py`用于启动程序以及其他的程序任务.

### Web框架一般都会有个统一的入口文件
一般Web框架都会具备一个统一的入口文件, 例如: PHP Web框架一般是`index.php`  

在入口文件, 初始化框架的时候我们可以加载一系列框架的功能函数, 或者对框架进行一些预加载, 缓存, 钩子等操作.  
通常, 路由的映射也是在此阶段完成的.  

由于Flask的高可定制性(项目的代码结构组织职能委托给用户), 这个入口文件是否具备也是用户决定  
一般我们会选择`__init__.py`作为入口文件, 当然你也可以自定义其他文件作为入口文件.

定义路由
--------

Request & Response
------------------

模板系统
--------

数据库操作
----------

非框架必备技能(Helpers)
-----------------------
