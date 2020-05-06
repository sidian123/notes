* 介绍

  Router用于在单页面程序中, 对不同组件进行路由的. 

  使用Router的好处有

  * 使用动态路由, 会生成多个Chunk, 减少打包的体积

* 安装

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

  