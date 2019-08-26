# 介绍

和Vue-CLI一类的脚手架, 用于快速搭建Vue应用.

--------

三种构建(或运行)模式

* Server Side Rendered: 先在**后端渲染**一遍, 获取异步数据填充页面, 再返回

  > 必须依赖前端的异步数据, 你是没办法渲染的吧

* Statically Generated: 生成静态页面, 将Js嵌入其中, 异步数据的**渲染交给前端**浏览器.

* Single Page App: 单页面程序, 只有一个页面, 多页面的实现全靠路由, 也是再前端渲染的.

-------

Nuxt.js脚手架包含下列内容:

- [Vue 2](https://vuejs.org/)
- [Vue Router](https://router.vuejs.org/en/)
- [Vuex](https://vuex.vuejs.org/en/) (included only when using the [store option](https://nuxtjs.org/guide/vuex-store))
- [Vue Server Renderer](https://ssr.vuejs.org/en/) (excluded when using `mode: 'spa'`(https://nuxtjs.org/api/configuration-mode))
- [vue-meta](https://github.com/nuxt/vue-meta)

使用[webpack](https://github.com/webpack/webpack) 加上[vue-loader](https://github.com/vuejs/vue-loader)和[babel-loader](https://github.com/babel/babel-loader) 去 bundle, code-split and minify your code.

# 使用

## 生成项目

生成

```bash
npx create-nuxt-app <project_name>
```

> 每次执行都会下载`create-nuxt-app`, 见[NodeJS/npx]

开发运行

```bash
npm run dev
```

生产环境

* 仅编译成Js, 且只能在Node中运行

  ```bash
  npm run build
  npm run start
  ```

* 编译出HTML, 可发布到任何服务器中

  ```bash
  nuxt generate
  ```

## 命令和发布

| COMMAND       | DESCRIPTION                                                  |
| :------------ | :----------------------------------------------------------- |
| nuxt          | Launch a development server on localhost:3000 with hot-reloading. |
| nuxt build    | Build your application with webpack and minify the JS & CSS (for production). |
| nuxt start    | Start the server in production mode (after running `nuxt build`). |
| nuxt generate | Build the application and generate every route as a HTML file (used for static hosting). |

> 我们使用的`npm dev`等命令, 执行的就是nuxt的命令.



# 项目结构

* `assets/`存放不被编译的静态文件, 如图片和字体等.

* `components/`存在组件

* `middleware/`编译前运行的脚本插件

* `plugins/`....emmm...

* `pages/`存放所有的视图(页面)和路由. 

  > 会为每个Vue页面产生一个路由?

* `static/`目录下的文件会被映射到url根目录下

* `store/`存放Vue Store, 该功能默认禁止.

* `layouts/`模板吧, `pages/`下的页面会放到模板中渲染.

  > 因此它的样式可以影响所有页面的样式

# 其他

## 全局和Scoped样式

布局`layouts/`的样式只在使用该布局的页面中共享, 全局的样式可以在所有布局中共享, 在配置文件的`css`属性中配置.

# 参考

* [入门视频](https://vueschool.io/courses/nuxtjs-fundamentals): 马马虎虎, 也就那样吧.

