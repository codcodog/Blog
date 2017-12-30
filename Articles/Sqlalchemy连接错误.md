Sqlalchemy 连接错误
===================

在使用 `Python` 的 `sqlalchemy` 库连接 `Mysql` 的时候遇到这样一个错误：  
> sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (1045, "Access denied for user 'root'@'119.137.52.47' (using password: YES)")

报这样的错很纳闷，因为在本地使用 `mycli` 和 `mysql-workbench` 都是可以连接上 `Mysql` 服务器的， 不存在 `帐号没权限`,`帐号密码不匹配` 的情况.

代码如下：
```Python
from sqlalchemy import *

from config import Config

class DB:

    def __init__(self):
        '''初始化，连接数据库
        '''
        DB_CONN = 'mysql+pymysql://{user}:{pwd}@{host}:{port}/{db}'.format(user=Config.user, pwd=Config.password, host=Config.host, db=Config.database, port=Config.port)
        engine = create_engine(DB_CONN, echo=True)
        self.conn = engine.connect()

if __name__ == '__main__':
    db = DB()
    print(db.conn)
```

Google了下，没有找到答案. 于是上 `IRC` 的 `#python` 频道询问，发现是因为自己的密码存在 `特殊字符` 导致的.  
`Sqlalchemy` 是使用 `database URL` 来创建连接的， 如果密码含有特殊字符的话， 则会被转译， 从而导致服务端认证无法通过，报 `Access denied for user` 这样的 `error`.

修改代码， 防止密码中的特殊字符被转译
```Python
from urllib.parse import quote

from sqlalchemy import *

from config import Config

class DB:

    def __init__(self):
        '''初始化，连接数据库
        '''
        DB_CONN = 'mysql+pymysql://{user}:{pwd}@{host}:{port}/{db}'.format(user=Config.user, pwd=quote(Config.password), host=Config.host, db=Config.database, port=Config.port)
        engine = create_engine(DB_CONN, echo=True)
        self.conn = engine.connect()

if __name__ == '__main__':
    db = DB()
    print(db.conn)
```

ok, 连接成功.
