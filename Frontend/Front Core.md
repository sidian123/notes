* 为啥使用路由

  在使用MVVC框架后, 页面体积会很大, 加载也会很慢

  引入路由后, 页面跳转间, 之前加载的组件等资源, 可被重复利用.

* 开发模式

  以前是多页面、前后端不分离的方式开发，并且前端页面都是通过后端模板视图输出的。

  如今流行前后端分离开发，狭隘来说，前后端仅通过Json通信。更广泛的，开发时解决跨域问题，基本不需要前端的干涉，只需定义接口接口即可。开发完成后，前后端可整合，也可分离部署。

  并且逐渐流行单页面开发，但通过url上的hash也能完成一个含“多页面”的系统，即单页面中包含了“多页面”，通过webpack+vue可以轻松胜任。

  > 案例：[vuejs 和 element 搭建的一个后台管理界面](https://www.cnblogs.com/taylorchen/p/6083099.html)

* 时间格式化

  [date-format](https://www.npmjs.com/package/date-format)

* 拷贝到剪贴板

  [v-clipboard](https://www.npmjs.com/package/v-clipboard)