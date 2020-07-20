#  Get start

* 介绍

  http server

* 依赖

  ```shell
  pip install Flask
  ```

* Demo

  `hello.py`

  ```python
  from flask import Flask
  app = Flask(__name__)
  
  @app.route('/')
  def hello_world():
      return 'Hello, World!'
  ```

* 运行

  ```python
app.run(port="8000", host="0.0.0.0", threaded=True, debug=True)
  ```
  
  * `debug` 是否进入debug模式, 该模式下, 当代码改变时, flask会自动重启服务. 默认`None`
* `host` 监听的ip, 默认localhost

# 路由

## 基础使用

使用装饰器`route()`绑定函数与URL

```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```

`methods`默认`[GET]`

## URL变量

### 介绍

获取url上的变量值, 也可以变量的转化器, 函数获取转化的值

```python
from markupsafe import escape

@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % escape(username)

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return 'Subpath %s' % escape(subpath)
```

### 转化器

转化器类型有:

| 转化器    | 描述|
| -------- | ------------------------------------------ |
| `string` | (default) accepts any text without a slash |
| `int`    | accepts positive integers                  |
| `float`  | accepts positive floating point values     |
| `path`   | like `string` but also accepts slashes     |
| `uuid`   | accepts UUID strings                       |

### 路由行为

1. url匹配路由, 若成功匹配, 则执行对应方法
2. 若匹配失败, 且有`/`后缀, 返回404
3. 若无`/`后缀, 则url路径加上`/`, 并重新匹配. 
   1. 若匹配不到, 则返回404; 
   2. 若匹配到了, 则重定向到有`/`的url上.

# Flask-SQLAlchemy

* 介绍

  Flask-SQLAlchemy将SQLAlchemy集成到了Flask, 并简化了SQLAlchemy的使用

* 安装

  ```shell
  pip install Flask-SQLAlchemy
  ```

  还需要安装驱动, 若用的mysql, 有多种驱动可供选择, 如`mysqlclient`
  
  ```shell
  pip install mysqlclient
  ```

## 连接&&配置

* url格式

  ```
  dialect[+driver]://user:password@host/dbname[?key=value..]
  ```

  * `dialect` 数据库名, 如mysql, oracle等
  * `driver` 驱动, 如`mysqldb`

* Engine

  可看作数据源, 支持多种数据库, 和连接池. 之后, 可直接通过Engine与数据库交互, 或者传入Session, 通过ORM来访问数据库.

  ![../_images/sqla_engine_arch.png](.Flask/sqla_engine_arch.png)

* 创建Engine

  ```
  engine = create_engine("mysql://scott:tiger@hostname/dbname",echo=True)
  ```

  * `encoding` 访问数据库用的编码, 默认`utf-8`

  * `echo` 是否打印执行的SQL, 默认`False`

  * `pool_size` 连接池内能支持存在的个数, 默认5

  * `max_overflow` 超出`pool_size`, 即溢出的连接个数, 默认10

    > 会话close后, 溢出的连接会直接关闭, 未溢出时才会回到连接池吧.

  * `pool_timeout` 获取连接的超时时长, 默认30

  * `pool_recycle` 连接池中连接的刷新时间, 默认`-1`, 即不刷新. 这可能会造成连接自动断开.

* 参考

  [Engine Configuration](https://docs.sqlalchemy.org/en/13/core/engines.html)

## 查询

* 语句描述
  * `Query.filter(*criterion)`

    * 支持复杂传参, 如 `==`, `>`

      ```python
      session.query(MyClass).filter(MyClass.name == 'some name', MyClass.id > 5)
      ```

      > 这应该是`Column`类的操作

    * 多个参数是`and`关系, 用`and_()`函数连接其他. 其他表达式, 如`or_()`, 需要显式使用

      ```python
      session.query(MyClass).filter(or_(MyClass.name == 'some name', MyClass.id > 5))
      ```

  * `Query.filter_by(**kwargs)`

    类似`filter()`, 但不支持`_and()`, `or_()` 以及表达式`>`, `==`等等. 略
  
  * `Column`
  
    声明Model对象时, 用的`Column`, 它代表字段, 提供了一些和查询相关的方法, 如`in_()`
  
    ```python
    property_list=db_session.query(SchemaDefinition) \
    .filter(and_(
        SchemaDefinition.schema_id == schema_id,
        SchemaDefinition.id.in_(property_ids),
        SchemaDefinition.category == 'link',
        SchemaDefinition.version != 'skip'
    )).all()
    ```
  
* 语句执行

  * `Query.first()` 查询, 获取第一条数据, 或返回`None`
  * `Query.all()` 查询, 返回所有记录

* 参考

  * [Query API](https://docs.sqlalchemy.org/en/13/orm/query.html)

  * [Column Elements and Expressions](https://docs.sqlalchemy.org/en/13/core/sqlelement.html#sqlalchemy.sql.expression.or_)



# 参考

* [Flask Docs](https://flask.palletsprojects.com/en/1.1.x/)

* [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/)
* [SQLAlchemy API](https://docs.sqlalchemy.org/en/13/genindex.html)

* [SQLALchemy之Python连接MySQL](https://blog.csdn.net/weixin_44080811/article/details/90030744)