# 介绍

NuxtJs和Vue-CLI是同类型的脚手架, 用于快速搭建Vue应用.

Nuxt.js脚手架包含下列内容:

- [Vue 2](https://vuejs.org/)

- [Vue Router](https://router.vuejs.org/en/)

  > 默认, NuxtJs会为`Pages/`下的每个Vue文件生成一个Route. 在Statically Generated模式下, 会被构建成多页面.

- [Vuex](https://vuex.vuejs.org/en/) (included only when using the [store option](https://nuxtjs.org/guide/vuex-store))

- [Vue Server Renderer](https://ssr.vuejs.org/en/) (excluded when using `mode: 'spa'`)

- [vue-meta](https://github.com/nuxt/vue-meta)

使用[webpack](https://github.com/webpack/webpack) 加上[vue-loader](https://github.com/vuejs/vue-loader)和[babel-loader](https://github.com/babel/babel-loader) 去 bundle, code-split and minify your code.

--------

三种构建(或运行)模式

* Server Side Rendered: 请求到来时, 可以在服务端渲染非用户相关的异步数据, 再返回结果.

  > 服务端获取异步数据的代码必须写在特定的函数中.
  >
  > 必须与NuxtJs的服务器配合使用

* Statically Generated: 构建生成静态多页面, 前端渲染.

* Single Page App: 单页面程序, 多页面靠路由实现, 同样前端渲染.

# 使用

## 项目开发

* 生成

    ```bash
    npx create-nuxt-app <project_name>
    ```

    > 每次执行都会下载`create-nuxt-app`, 见[NodeJS/npx]

* 开发运行

    ```bash
    npm run dev
    ```

    > 运行于SSR模式上.

* 生产环境

    * SSR模式, 且只能在Node中运行

      ```bash
      npm run build
      npm run start
      ```

    * 多页面模式

      ```bash
      nuxt generate
      ```
      
    * 单页面模式
    
      * 先修改配置文件`nuxt.config.js`的`mode`属性为`spa`
      * 构建`npm run build`
      * 然后运行, `npm run start`这样?

## 命令

| COMMAND       | DESCRIPTION                                                  |
| :------------ | :----------------------------------------------------------- |
| nuxt          | Launch a development server on localhost:3000 with hot-reloading. |
| nuxt build    | Build your application with webpack and minify the JS & CSS (for production). |
| nuxt start    | Start the server in production mode (after running `nuxt build`). |
| nuxt generate | Build the application and generate every route as a HTML file (used for static hosting). |

> 我们使用的`npm run dev`等命令, 执行的就是nuxt的命令, 见`package.json`文件

# 项目结构

* `assets/`存放不被编译的静态文件, 如图片和字体等.

* `static/`目录下的文件会被映射到url根目录下

* `components/`存放组件的地方

* `pages/`存放所有的视图(页面)和路由. 

  > 会为每个Vue页面产生一个路由

* `layouts/`布局模板, `pages/`下的页面会放到模板中渲染. 所有页面默认使用`default`布局.

* `store/`存放Vuex Store(状态的容器), 该功能默认禁止.

* `middleware/`编译前运行的插件(函数)

* `plugins/`Vue实例运行前的插件(函数)

nuxtjs提供了方便访问这些目录的别名(仅用于模块请求)

别名|目录
----|-----
`~`或`@`|`srcDir`
`~~`或`@@`|`rootDir`

默认`srcDir`与`rootDir`一样, 指向项目根路径, 可以自行配置.

# 资源访问

## 模块依赖

NuxtJs会自动处理样式和Vue模板中的url, 解析为模块依赖, 由对应的loader加载.

> 如`<img src="...">`,`background:url(...)`,`@import`中的url会被处理

一般的模块依赖在Webpack打包时, 会打包到一块去. 对于较小的二进制文件也会被打包到一个文件中, 大文件则只能放到文件中.

> 如`<img src="~/assets/image.png">`或转化为`<img src=""/_nuxt/img/image.0c61159.png">`

一般资源文件放入`assets/`目录下.

## static/

指向`static/`目录中资源文件的url不会被当作模块依赖. 打包时会被移到根目录下, 因此可以通过根路径来访问.

# 插件

NuxtJs中所谓的插件就是在页面的Vue实例生成之前, 执行的一段代码.

使用步骤:

1. 在`plugins/`下建立Js文件, 如

   ```javascript
   import Vue from 'vue';
   import ElementUI from 'element-ui';
   //样式也可以在配置文件的css属性中配置,都一样
   import 'element-ui/lib/theme-chalk/index.css';
   
   Vue.use(ElementUI);
   ```

2. 在`nuxt.config.js`的`plugins`中指定插件位置. 如

   ```JavaScript
   plugins: [
       '@/plugins/element-ui'
   ]
   ```

   > 只有在这声明了才会被使用

# 配置

## 代理

通过代理解决跨域问题.

首先安装依赖

```bash
npm i @nuxtjs/axios @nuxtjs/proxy -D
```

在`nuxt.config.js`文件中

```java
modules: [
    '@nuxtjs/axios',
    '@nuxtjs/proxy'
],
proxy: {
    '/api': {
        target: 'http://example.com',
        pathRewrite: {
            '^/api' : '/'
        }
    }
}
```

>注意点
>
>* 以下选项默认为`true`
>  * `changeOrigin`: 是否设置`host`头字段为target url
>  * `ws`: 是否代理websockets
>  * `pathRewrite`的key可以使用正则
>
>* 匹配问题
>  * 顺序匹配, 先匹配的有效. 即`/api`后的`/api2`永远不会被匹配.
>  * 注意重写, `'/api'`最好重写为`''`, `'/api/'`重写为`'/'`

# 其他

## 元数据head

* 在配置文件`nuxt.config.js`中配置`head`属性, 作用域所有页面
* 在`pages/`下的单个Vue文件中配置`head`属性, 仅作用于当前页面.

> 还有其他tricks, 略.

## 抽离公共模块

对于模块依赖, Webpack打包时会将通用的公共部分抽离出来, 有利于浏览器缓存.

当自动抽离不如人意时,  如分离外部模块, 可以手动抽离.

1. 将文件移入`static/`目录下

   > 为了避免被当作模块依赖而被打包

2. 在`nuxt.config.js`配置全局`head`, 或在`pages/`下的Vue文件中为单个页面配置`head`. 链接到所使用的模块, 如css:

   ```javascript
   head: {
       title: 'MyWork',
       link: [
           {rel: 'stylesheet', href:   '/semantic.css'}
       ]
   }
   ```

## 全局和Scoped样式

布局`layouts/`的样式只在使用该布局的页面中共享, 全局的样式可以在所有布局中共享, 在配置文件的`css`属性中配置.

## 单页面

Vue-CLI中的单页面需要服务器的配置, 将所有Url的路由引向单页面.

而NuxtJs的实现不同, 它构建出了多页面, 但使用**同样的Js入口**. 这样无需服务器的配合了.

# 参考

* [入门视频](https://vueschool.io/courses/nuxtjs-fundamentals): 马马虎虎, 也就那样吧.

