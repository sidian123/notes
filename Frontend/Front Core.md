# 为啥使用路由

在使用MVVC框架后, 页面体积会很大, 加载也会很慢

引入路由后, 页面跳转间, 之前加载的组件等资源, 可被重复利用.

# 跨域

* 介绍

  当访问的资源与当前页面的协议, 端口和域名任意一个不同时, 都是跨域访问. 这种行为在浏览器中是被禁止的.

* 跨域方案
  * CORS（cross domain resources share）：设置http头部，允许跨域资源共享
    - 插件：浏览器端拦截并设置头部
  * Jsonp：使用script标签获取数据。原理：script标签不受同源政策约束
  * 代理：同源政策仅存在于浏览器，因此可通过服务端代理http请求。如nginx的方向代理。
  * 模拟数据：前端使用模拟数据开发，不直接跨域访问。如使用库`mock.js`
  
* 遇到的问题

  Https的页面中访问Http的API接口, 后端开启了CORS, 仍报错Mixed Block. 

  其实可以跨域, 不过浏览器此时将HTTP的内容视为不安全的, 因此又加了一层内容拦截.

  在地址栏中, 允许不安全内容的访问即可

  ![image-20200108145158177](.Front%20Core/image-20200108145158177.png)
  
  > 如果访问是本地localhost的HTTP内容, 是允许的. 但在火狐中不允许, 因为这是W3标准, 见[Reason: CORS request did not succeed](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSDidNotSucceed)

# 开发模式

以前是多页面、前后端不分离的方式开发，并且前端页面都是通过后端模板视图输出的。

如今流行前后端分离开发，狭隘来说，前后端仅通过Json通信。更广泛的，开发时解决跨域问题，基本不需要前端的干涉，只需定义接口接口即可。开发完成后，前后端可整合，也可分离部署。

并且逐渐流行单页面开发，但通过url上的hash也能完成一个含“多页面”的系统，即单页面中包含了“多页面”，通过webpack+vue可以轻松胜任。

案例：[vuejs 和 element 搭建的一个后台管理界面](https://www.cnblogs.com/taylorchen/p/6083099.html)

