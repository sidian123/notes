# 开始

## 介绍

Vue Router能让单页面程序在不同组件间路由.

Router的特性有:

* 使用动态路由, 会生成多个Chunk, 减少打包的体积
* 路由时, 支持视图过渡的特效
* ...

## 安装

包下载

```shell
npm install --save vue-router
```

将Router注册为Vue插件

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

# 使用

## 映射关系建立

### 使用概念

**路由**建立了**组件**和**视图**之间的路由关系, 即每个URL匹配模式下, 都对应一个组件与视图的确认关系.

路由与视图有层级关系, 且一一对应; 同层上的多个视图, 与同层的路由中的组件有一一对应关系.

### 入门使用

```vue
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <!-- 视图 -->
  <router-view></router-view>
</div>
```

```javascript
//配置路由
const router = new VueRouter({
  routes:[
      { path: '/foo', component: Foo },
      { path: '/bar', component: Bar }
    ]
})
//挂载路由到根实例上
const app = new Vue({
  router
}).$mount('#app')
```

在组件内, 可以通过`this.$router`获取上述生成的路由实例; 通过`this.$route`的属性, 可以获取当前路由的信息.

### 嵌套路由&视图

即路由和视图都有层级关系, 且路由与视图在层级上一一对应.

路由, 一共两层

```javascript
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // UserProfile will be rendered inside User's <router-view>
          // when /user/:id/profile is matched
          path: 'profile',
          component: UserProfile
        },
        {
          // UserPosts will be rendered inside User's <router-view>
          // when /user/:id/posts is matched
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})

```

顶层视图

```html
<div id="app">
  <router-view></router-view>
</div>
```

第二层视图, 在User组件中

```javascript
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```

### 多视图

同层上的多个视图, 与同层的路由中的组件有一一对应关系

多个视图, 无`name`的视图, 默认`default`

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

与同层的路由的某个路由的组件一一对应

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

> 注意, 是`components`, 而非`component`

## 动态路由匹配

* 动态片段

    即匹配模式中的某个片段可以任意匹配URL片段.
    
    下面的`id`匹配任何URL片段, 如`/user/foo`, `/user/bar`都将路由到`User`组件上.
    
```javascript
    const User = {
      template: '<div>User {{ $route.params.id }}</div>'
    }
    
    const router = new VueRouter({
      routes: [
        // dynamic segments start with a colon
        { path: '/user/:id', component: User }
      ]
    })
```

    `$route.params`可以取出动态片段的内容, 并展示为一个对象, 如`{id:'foo'}`

* `*` 匹配任何

    ```js
    {
      // will match everything
      path: '*'
    }
    {
      // will match anything starting with `/user-`
      path: '/user-*'
    }
    ```

    `this.$route.params.pathMatch`可以获取到`*`匹配到的内容

* 正则匹配

    `vue-router`内部用到了[path-to-regexp](https://github.com/pillarjs/path-to-regexp/tree/v1.7.0)[ ](https://github.com/pillarjs/path-to-regexp/tree/v1.7.0)来完成路由匹配, 除了上述功能, 还提供了正则匹配, 略.

* 匹配顺序

    被多个路由匹配, 路由声明顺序越靠前, 优先级越高.

## API

### 信息获取

`route`对象表示当前激活的路由状态, 含有当前解析过的URL信息, 和URL匹配记录. 在组件中可以通过`this.$route`访问到.

常用属性如下

* `$route.path` 绝对路径地址, 如`/foo/bar`
* `$route.params` 动态路由时匹配到的数据, 以对象的形式提供. 默认空对象
* `$route.query` URL上的请求参数, 对象形式. 默认空对象
* `$route.hash` 路径上的hash(含`#`), 不是url上的哈希. 如`http://localhost:8080/#/a/c#abc` , 则hash为`#abc`

* ...

### 路由跳转

路由跳转功能由路由实例提供, 可通过`this.$router`获取该实例.

* `router.push(location, onComplete?, onAbort?)`

  路由跳转, 产生历史记录.

   `location`可以是字符串, 可以是对象

  ```javascript
  // literal string path
  router.push('home')
  
  // object
  router.push({ path: 'home' })
  
  // named route. 可以直接跳转到命名路由上, 即使它是子路由
  router.push({ name: 'user', params: { userId: '123' } })
  
  // with query, resulting in /register?plan=private
  router.push({ path: 'register', query: { plan: 'private' } })
  ```

  > 注意, `path`属性与`params`冲突.

  该方法等同于`route-link`组件

  | Declarative              | Programmatic       |
  | ------------------------ | ------------------ |
  | `<router-link :to="..."` | `router.push(...)` |

  > 路由跳转的两种不同使用方式

* `router.replace(location, onComplete?, onAbort?)`

  替换路由, 不产生历史. 详细使用与上述一致.

* `router.go(n)`

  前进或后退历史记录

  ```javascript
  // go forward by one record, the same as history.forward()
  router.go(1)
  
  // go back by one record, the same as history.back()
  router.go(-1)
  
  // go forward by 3 records
  router.go(3)
  
  // fails silently if there aren't that many records.
  router.go(-100)
  router.go(100)
  ```

## 命名路由&视图

可以通过路由名字, 直接找到路由, 实现跳转, 即使是子路由. 如

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

声明式跳转

```html
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

编程式跳转

```js
router.push({ name: 'user', params: { userId: 123 }})
```

-------------

命名的视图, 当然是用于同一层存在多个视图, 路由过程中好确定哪个组件路由到哪个视图上. 见上述*映射关系/多视图* 小节.

## HTML5历史模式

默认使用`hash`模式, 来模式页面跳转和历史记录. 当然, Vue Router也提供HTML自带的历史记录模式, 如

```javascript
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

还需要配置服务器, 因为Vue单页面本质只有一个页面, 若访问其他路径, 将直接报404. 为避免该情况的发生, 应在服务器端配置, 所有URL都指向Vue单页面. 如Nginx的配置

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

> Nuxtjs即使没有使用配置Nginx, 也能使用HTML5历史模式. 因为它提供了多个单页面入口. 

# 进阶

## 监听&生命周期&守卫

```js
const User = {
  template: '...',
  watch: {
    $route(to, from) {
      // react to route changes...
    }
  }
}
```

```js
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

## 动态路由(懒加载)

也就是使用异步组件作为路由的映射

```javascript
const router = new VueRouter({
  routes: [
    { path: '/foo', component: ()=>import('./Foo.vue') }
  ]
})
```

# 其他

## Demo

```javascript
{
    path: 'drug-ingredient-category',
    name: 'DrugIngredientCategory',
    meta: {
    title: '药物成分分类'
    },
    component: () => import('../views/DrugManage/DrugIngredientCategory/DrugIngredientCategory')
}
```

# 参考

* [Vue Router](https://router.vuejs.org/)
* [Vue Router API](https://router.vuejs.org/api)























