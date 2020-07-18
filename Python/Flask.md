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

  * 方式一

    ```shell
    export FLASK_APP=hello.py; flask run
    ```

  * 方式二

    无`FLASK_APP`变量时, `flash`会寻找当前目录下`wsgi.py` 或 `app.py`文件, 并运行.

    ```shell
    flask run
    ```

  * 运行选项

    * 允许被其他人访问

      ```shell
      flask run --host=0.0.0.0
      ```

    * 开发模式

      该模式下, 当代码改变时, flask会自动重启服务.

      ```shell
      $ export FLASK_ENV=development
      $ flask run
      ```

  * 方式三

    ```python
    app.run(port="8000", host="0.0.0.0", threaded=True, debug=False)
    ```

    

* 参考

  [Flask Docs](https://flask.palletsprojects.com/en/1.1.x/)