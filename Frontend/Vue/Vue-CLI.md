[TOC]

# 一 单文件组件

全局组件的缺点：

* 全局定义造成名字污染
* 模板字符串没有语法高亮

* 没有css支持，即css不能与组件放在一块
* 没有构建工具，只能是有HTML和ES5，且浏览器支持可能不够好。

vue提供Vue CLI3工具，其实就是已配置好的webpack。解决了上述问题，能够将组件写到单个文件中、且支持预处理器（即可使用高级语法、TypeScript、Scss等等）、支持热加载（也启动了HMR？）和预编译。

> 如果webpack玩的六，可使用vue-loader自行搭配webpack，否则老老实实的使用vue-cli

vue-cli的配置通常符合我们的需求，但也提供配置入口，让我们微调，参考：[Configuration Reference](<https://cli.vuejs.org/config/#global-cli-config>)

> 给我感觉，就像spring boot似的，以预配置好了，我们可直接专注于开发。

# 二 组成部分

* CLI：全局安装，提供`vue`命令。
  * 用于快速创建vue项目（`vue create`）；
  * 甚至直接执行组件进行快速原型设计（`vue serve`）；
  * 也提供图形界面管理项目（`vue ui`）
* CLI Service：当创建项目时，局部安装到项目的开发依赖。构建于webpack和webpack-dev-server之上，含：
  * 加载其他CLI插件的核心服务
  * 针对于大多数应用优化的内部webpack配置。
  * 含有命令`serve`,`build`和`inspect`。使用`vue-cli-service`运行，如`npx vue-cli-service serve`，但vue项目为你配置了默认的脚本，可使用npm运行，如`npm serve`。

* CLI Plugins：vue项目可选的插件，用于提供更强大的功能。

# 三 安装

```bash
#安装vue cli
sudo npm install -g @vue/cli
```

# 四 基础

## 快速原型设计

```bash
# 为了支持快速原型设计
sudo npm install -g @vue/cli-service-global
```

### vue serve

只需一个`.vue`或`.js`文件，`vue serve`直接运行文件到开发模式下，使用与`vue create`创建的项目一样的配置。

```
Usage: serve [options] [entry]

serve a .js or .vue file in development mode with zero config

Options:

  -o, --open  Open browser
  -c, --copy  Copy local url to clipboard
  -h, --help  output usage information
```

默认入口为`main.js`, `index.js`, `App.vue` or `app.vue`，也可自己指定：
```bash
vue serve MyComponent.vue
```



如果需要，还可提供`index.html`,`package.json`、安装和使用局部依赖、配合vue配置文件等等。

### vue build

```
Usage: build [options] [entry]

build a .js or .vue file in production mode with zero config


Options:

  -t, --target <target>  Build target (app | lib | wc | wc-async, default: app)
  -n, --name <name>      name for lib or web-component (default: entry filename)
  -d, --dest <dir>       output directory (default: dist)
  -h, --help             output usage information
```

构建目标文件，生产可用于生产的打包文件，也可构建成library或组件。

## 创建项目

运行命令：

```bash
vue create hello-world
```

然后选择默认设置或手动设置。在创建项目的过程中，你的一些设置会被保存在`~/.vuerc`中。

使用GUI创建项目：`vue ui`；老版本vue创建项目的版本：`vue init webpack my-project`

# 五 插件

vue cli使用基于插件的架构，依赖大多被组织成`@vue/cli-plugin-`的形式。创建能够修改内部webpack的配置，与注入命令到`vue-cli-service`中。创建项目时，被列出的很多功能，都是被默认插件体用的。

添加新插件时，使用`vue add`。插件其实就是node的package，为啥不用`npm install`呢？因为使用`vue add`时会通在vue项目中配置该插件。

# 六 CLI Service

vue cli项目中，会同时安装可执行文件`vue-cli-service`，它是用来运行、构建vue项目的。

* `vue-cli-service serve`：开启一个dev server（基于wepack-dev-server），并开HMR功能（热模块替换）。
* `vue-cli-service build`：产生一个生产级的bundle

等等

# 七 HTML和静态资源

* `public/index.html`作为[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)使用的模板
  
  * [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)主要用于为你的bundle创建一个简单的html文件来运行bundle（即自动注入你bundle）。
* 模板中可插入一些占位符，略。
  
* **多页面app**：需在`vue.config.js`配置`pages`选项。它能够有效的共享通用chunks

* **public目录**：该目录下的文件会被简单的拷贝到dist中（index.html除外，它被注入了bundle），需要使用绝对路径来访问。发布时需要考虑路径问题，因此所需资源因放在src下

* **静态资源**：打包时对JavaScript、css、.vue文件中**指向资源**的url的处理：

  * `/`：绝对路径，如`/image/foo.png`，不做处理。
  
  * `./` 或 `../`：相对路径，会被解析成模块请求，最终转化为根绝对路径

  * `@`：指向`<projectRoot>/src`，会被解析成模块请求，最终转化为根绝对路径。
  
  * `~`：指向`<projectRoot>/node_modules`，如`~some-npm-package/foo.png`，会被解析成模块请求，最终转化为根绝对路径。
  
    > 经测试，暂时这个没屁用
  
  对资源文件的模块请求转化为根绝对路径时，添加前[publicPath](<https://cli.vuejs.org/config/#publicpath>)，默认`/`，可根据项目修改，或者设置为`''`或`./`，即转化为相对路径。

# 八 css
## Vue Loader
- Vue Loader用于加载`.vue`文件。

- `<template>`中的URL会被转化为webpack模块请求。使用规则与第七章一致，因此`dog.png`文件的URL为`./dog.png`

- 如果使用css预处理器，先安装依赖：

  ```bash
  npm install -D sass-loader node-sass
  ```

  然后在`style`元素的`lang`属性上指定语言，如：

  ```html
  <style lang="scss">
  /* write SCSS here */
  </style>
  ```



## Scoped css

### 介绍

当`style`元素加上`scoped`属性时，组件的css只会作用到该组件的单个实例上。它是通过**后css处理器**实现的。

### 原理

css处理前：
```html
<style scoped>
.example {
  color: red;
}
</style>

<template>
  <div class="example">hi</div>
</template>
```
处理后：
```html
<style>
.example[data-v-f3f3eg9] {
  color: red;
}
</style>

<template>
  <div class="example" data-v-f3f3eg9>hi</div>
</template>
```
可以看出，它改动了两点：
1. html元素上添加了一个该组件实例唯一标识的属性`data-v-f3f3eg9`
2. css选择器上添加了属性选择器，让css选择唯唯一时，还加重了它的[重要性](https://blog.csdn.net/jdbdh/article/details/84317656#242Specificity_335)

因此父组件想影响有`scoped`的子组件的样式时，需要设法加重选择器的重要性，比如`!important`

### 注意点

* 组件的不同的实例的css样式都不重复，会导致解析速度下降。
* 对于动态生产的组件不会起作用，因为scoped是在打包时作用的。
* 编写容器组件（包括含`<slot>`的组件）时，容器组件的根class不要与子组件的根class同名，因为子组件的根class仍会拥有容器组件的唯一标识属性，导致子组件样式被覆盖。因此，最好容器组件的class唯一或选择器唯一。

### 样式穿透

有时候, 需要在scope样式中, 作用于第三方组件, 即样式穿透到第三方组件.

* 在含scope的sass/scss中, 用`::v-deep`

  ```scss
  ::v-deep .child-class {
      background-color: #000;
  }
  ```

* 除此之外, 如css, 可使用`>>>`

  ```css
  >>> .child-class {
      background-color: #000;
  }
  ```

> 至于`/deep` 是被弃用的

### 参考

* [Scoped CSS](https://vue-loader.vuejs.org/guide/scoped-css.html#mixing-local-and-global-styles)
* [How do I use /deep/ or >>> in Vue.js?](https://stackoverflow.com/questions/48032006/how-do-i-use-deep-or-in-vue-js)

# 九 vue配置

* 配置文件`vue.config.js`
* 参考：[Configuration Reference](<https://cli.vuejs.org/config/>)

## pages

构建vue成多页面模式，默认单页面。文档介绍很详细：[pages](<https://cli.vuejs.org/config/#pages>)，思路就是指定入口文件和html模板。貌似vue会为你抽离通用chuncks？？被访问过的页面会被缓存起来。



## devServer

配置自带的devServer服务器

* port：设置端口号，默认8080

* proxy：devServer代理转发请求，主要用于跨域，配置如下：

  ```javascript
      devServer:{
        //devServer监听的端口
        port:8000,
        //配置代理
        proxy:{
          //拦截的url, 匹配前缀,支持正则
          '/api':{
            //转发到目标服务器的url
            target:"http://localhost:8080",
            //是否代理websockets,可选
            ws:true,
            //是否修改Host头部，可选
            changeOrigin:true,
            //修改请求路径,支持正则
            pathRewrite:{'/api':''},
            //是否忽略掉Https证书问题
            //secure: false
          }
        }
    }
  ```
  
  > 由于是服务器代理请求的，因此没有同源政策的限制。
  
  > 一般目的URL=target+拦截的url, 若存在路径修改, 则=target+修改过的url
  
  > **注意**
  >
  > URL最先匹配的代理规则会被使用.
  >
  > 如, 依顺序存在规则`/api`, `/api2`, 那么`/api2`规则永远不会被使用, 应该改成`/api/`, `/api2/`
  
  > 参考
  >
  > * [devServer.proxy](https://cli.vuejs.org/config/#devserver-proxy)
  > * [context matching](https://github.com/chimurai/http-proxy-middleware#context-matching)

## eslint

eslint帮助减少隐藏错误.

禁用的多种方法:

1. 禁止一行eslint校验

    ```javascript
    javascript code // eslint-disable-line
    ```
    
2. 禁止多行的eslint校验

    * `eslint-disable` 禁止该行下面的eslint校验
    * `eslint-enable` 启动该行下面的eslint校验

    例子:

    ```javascript
    /* eslint-disable */
    your code
    /* eslint-enable */
    ```

3. 禁用eslint, 有多种方案

    1. 如果用的vue-cli2, 在`config/index.js`中, 添加

        ```javascript
        useEslint: false
        ```

    2. 如果用的vue-cli3, 在`.eslintrc.js`中设置

        ```javascript
        root: false
        ```

    3. 移除`@vlue/cli-plugin-eslint`包

    4. 将要忽略目录或文件添加到`.eslintignore`文件

        ```
        myfile.js
        *.test.js
        data/*
        /build/
        /config
        /test/unit/coverage/
        ```
        
        > 上面提供了多种写法, 其中相对路径是相对于`.eslintignore`文件的

> 参考：
>
> * [@vue/cli-plugin-eslint](<https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint>)
> * [How can I disable eslint correct?](https://stackoverflow.com/questions/58634424/how-can-i-disable-eslint-correct)

## 模式&环境变量

* 不同的模式下, 会读不同的配置文件, 文件中可以配置环境变量

  ```
  .env                # loaded in all cases
  .env.local          # loaded in all cases, ignored by git
  .env.[mode]         # only loaded in specified mode
  .env.[mode].local   # only loaded in specified mode, ignored by git
  ```

  > `*.local`的文件在vue脚手架的`.gitignore`文件中默认忽略.

  `env`和`env.local`所有情况下都读, 但变量优先级要比模式具体相关的文件要低.

* 配置文件的一个Demo, `.env.staging`

  ```
  NODE_ENV=production
  VUE_APP_TITLE=My App (staging)
  ```

* `NODE_ENV`一般等于模式值, 如上述的`staging`. 但一般还是会在配置文件中修改`NODE_ENV`为**特殊**的三个值, 不同值时, 会有不同的插件执行, 如

  * `development` 将启动热模块更新HMR. 使用`vue-cli-service- serve`命令时默认该值
  * `test `为单元测试进行优化. 使用`vue-cli-service test:unit`命令时默认该值
  * `production` 为生成环境优化. 使用`vue-cli-service build`和`vue-cli-service test:e2e`时默认该值

* 环境变量使用

  ```
  console.log(process.env.VUE_APP_NOT_SECRET_CODE)
  ```

  上述变量会在构建时**替换**.

* 环境变量其他设置方式

  在`vue.config.js`中配置

  ```js
  process.env.VUE_APP_VERSION = require('./package.json').version
  
  module.exports = {
    // config
  }
  ```

> [Modes and Environment Variables](https://cli.vuejs.)/guide/mode-and-env.html#modes

# 十 其他

## Vue Devtools

浏览器插件，可以查看vue组件的属性，仅在开发模式下，该插件才会生效。

## Hot Reload

当你编辑页面时，Hot Reload不是简单的重载页面，而是置换被编辑的组件（无需重载），甚至能保存被置换组件的状态。这样当你更改样式、模板时不必担心组件状态。

未完待续，参考：[hot Reload](<https://vue-loader.vuejs.org/guide/hot-reload.html>)

## 入口

貌似默认main.js

# 十一 bug
使用过程中遇到的莫名其妙的bug：
1. npm在wsl貌似有权限错误，解决：
	```bash
	npm config set unsafe-perm true
	```
2.  核心模块错误（core-js module error）。解决：修改`babel.config.js`文件，添加选项：
	```bash
	presets: [ [ "@vue/app", { useBuiltIns: "entry" } ] ]
	```

# 参考

* [Vue Cli Guide](https://cli.vuejs.org/guide/#components-of-the-system)
* [Vue Cli Config](https://cli.vuejs.org/config/)