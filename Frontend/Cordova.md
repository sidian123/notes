# 引言

* 介绍

  提供以web技术编写移动端应用的功能.

* 原理

  将web页面嵌入到APP中, 通过WebView渲染, 同时以插件的方式将原生系统组件暴露出来

* 使用方式

  * 打包成完整APP
  * 仅嵌入到原生APP中

# 基础使用

* 安装Cordova

  ```shell
  npm install -g cordova
  ```

* 新建项目

  ```shell
  cordova create MyApp
  ```

* 添加目标平台

  ```shell
  cordova platform add browser
  ```

  > 除此之外, 还有ios, android

* 运行browser

  ```shell
  cordova run browser
  ```

* 查看支持/已安装的平台

  ```shell
  cordova platform ls
  ```

# 环境安装

* 构建之前, 必须安装对应平台的SDK. 其中, `browser`无需任何SDK.

  检查平台构建的必备条件

  ```shell
  cordova requirements
  ```

* 