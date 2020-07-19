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

  





























# 参考

* [Flask Docs](https://flask.palletsprojects.com/en/1.1.x/)

* [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/)
* [SQLAlchemy API](https://docs.sqlalchemy.org/en/13/genindex.html)