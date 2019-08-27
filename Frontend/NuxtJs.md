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

# Assets

# 配置



# 其他

## 全局和Scoped样式

布局`layouts/`的样式只在使用该布局的页面中共享, 全局的样式可以在所有布局中共享, 在配置文件的`css`属性中配置.

# 参考

* [入门视频](https://vueschool.io/courses/nuxtjs-fundamentals): 马马虎虎, 也就那样吧.

