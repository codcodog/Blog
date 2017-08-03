Flask 学习
==========

目录
----
- [项目结构](#项目结构)
    + [项目的目录结构是如何定义的](#项目的目录结构是如何定义的)
    + [是否具备入口文件](#是否具备入口文件)
- [定义路由](#定义路由)
    + [如何定义一个URL](#如何定义一个url)
- [Request & Response](#request--response)
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
- [非框架必备技能(Helpers)](#非框架必备技能helpers)
    + 文件上传
    + Log日志
    + 发送Email
    + 图片处理

项目结构
--------

### 项目的目录结构是如何定义的
> 由于Flask把代码的组织职能委托给用户, 所以项目的目录结构一般是用户根据自己的需要而实现.

大型程序的目录结构(参考)
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

### 是否具备入口文件
一般Web框架都会具备一个统一的入口文件, 例如: PHP Web框架一般是`index.php`  

在入口文件, 初始化框架的时候我们可以加载一系列框架的功能函数, 或者对框架进行一些预加载, 缓存, 钩子等操作.  

通常, 路由的映射也是在此阶段完成的.  

由于Flask的高可定制性(项目的代码结构组织职能委托给用户), 这个入口文件是否具备也是用户决定  

一般我们会选择`__init__.py`作为入口文件, 当然也可以自定义其他文件作为入口文件.

定义路由
--------
> 处理URL和函数之间关系的程序称为路由.

### 如何定义一个URL
- 利用`route()`装饰器, 把装饰的函数注册为路由.

```
@app.route('/')
def index():
    return 'Index Page'
```

- 动态路由

规则: `<converter:variable_name>`

```
@app.route('/user/<username>')
def show_user_profile(username):
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return 'Post %d' % post_id
```

转换器:  
> int: 接受整数  
> float: 同int, 但是接受浮点数  
> path: 和默认的相似, 但也接受斜线  

- 唯一URL / 重定向行为

> Flask的URL 规则基于Werkzeug的路由模块. 这个模块背后的思想是基于Apache以及更早的HTTP服务器主张的先例,保证优雅且唯一的URl.

```
@app.route('/project/')
def project():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```

虽然它们看起来相似, 但注意它们结尾斜线的使用在URL定义中不同. 

第一种情况, 指向project的规范URL尾端有一个斜线. 这种感觉很像在文件系统中的文件夹.  
访问一个结尾不带斜线的URL会被Flask重定向到带斜线的规范URL中去.

然而, 第二种情况的URL结尾不带斜线, 类似 UNIX-like 系统下的文件的路径名. 访问结尾带斜线的URL, 会
产生一个404 "Not Found" 的错误.

这个行为使得在遗忘尾斜线时, 允许关联的URL接任工作, 与Apache和其他服务器的行为并无二异. 此外, 也保证了
URL的唯一, 有助于避免搜索引擎引同一个页面两次.

- 构造URL

> 使用 `url_for()` 来给指定的函数构造URL.

```
>>> with app.test_request_context():
...  print url_for('index')
...  print url_for('profile', username='John Doe')
...
/
/user/John%20Doe
```
- HTTP 方法

```
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```

- 蓝图(blueprints)

> Flask 用 *蓝图* 的概念来在一个应用中或跨应用制作应用组件和支持通用的模式.  
> 一个Blueprint对象与Flask应用对象的工作方式很像, 但它确实不是一个应用, 而是一个描述如何构建或扩建应用的*蓝图*.  
> 在`蓝图`中定义的路由处于休眠状态, 直到`蓝图`注册到程序上后, 路由才真正成为程序的一部分.  

```
from flask import Blueprint, render_template, abort
from jinja2 import TemplateNotFound

simple_page = Blueprint('simple_page', __name__,
                        template_folder='templates')

@simple_page.route('/', defaults={'page': 'index'})
@simple_page.route('/<page>')
def show(page):
    try:
        return render_template('pages/%s.html' % page)
    except TemplateNotFound:
        abort(404)
```

注册蓝图: 
```
from flask import Flask
from yourapplication.simple_page import simple_page

app = Flask(__name__)
app.register_blueprint(simple_page)
```

详细了解:  

[用蓝图实现模块化的应用](http://docs.jinkan.org/docs/flask/blueprints.html)  
[sopython.com](https://github.com/sopython/sopython-site)  

(PS: 查看`sopython.com`项目源码, 可以了解到大型项目中如何使用`蓝图`构建应用)


Request & Response
------------------

模板系统
--------

数据库操作
----------

非框架必备技能(Helpers)
-----------------------
