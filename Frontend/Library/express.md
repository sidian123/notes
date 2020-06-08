* 安装

  ```shell
  npm install express --save
  ```

* hello world

  ```javascript
  const express = require('express')
  const app = express()
  const port = 3000
  
  app.get('/', (req, res) => res.send('Hello World!'))
  
  app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
  ```

* 脚手架

  ```shell
  npx express-generator
  ```

  看看就好, 就是一个有路由, 视图模板的Demo

* 路由

  * 调用形式

      ```javascript
      app.METHOD(PATH, HANDLER)
      ```
    
      其中
    
      - `app` is an instance of `express`.
      - `METHOD` is an [HTTP request method](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods), in lowercase.
      - `PATH` is a path on the server.
      - `HANDLER` is the function executed when the route is matched.

  * 例子

    ```javascript
    app.get('/', function (req, res) {
      res.send('Hello World!')
    })
    ```
  
    ```javascript
    app.post('/', function (req, res) {
      res.send('Got a POST request')
    })
    ```
  
    ```javascript
    app.put('/user', function (req, res) {
      res.send('Got a PUT request at /user')
    })
    ```
  
    ```javascript
    app.delete('/user', function (req, res) {
      res.send('Got a DELETE request at /user')
    })
    ```
  
* 静态资源服务

  让客户端能访问到静态资源, 如img, css, js等文件

  ```javascript
  // 资源映射到根路径下
  app.use(express.static('public'))
  app.use(express.static('files'))
  // 资源映射到/static下
  app.use('/static', express.static('static'))
  ```

  上述定义了三个目录作为静态资源, 资源查找时, 先定义的目录优先级高.

  访问`public`或`files`目录下的文件

  ```
  http://localhost:3000/images/kitten.jpg
  http://localhost:3000/css/style.css
  http://localhost:3000/js/app.js
  http://localhost:3000/images/bg.png
  http://localhost:3000/hello.html
  ```

  访问`static`目录下的资源, 有前缀

  ```
  http://localhost:3000/static/images/kitten.jpg
  http://localhost:3000/static/css/style.css
  http://localhost:3000/static/js/app.js
  http://localhost:3000/static/images/bg.png
  http://localhost:3000/static/hello.html
  ```

* 参考

  [Express](http://expressjs.com/)

