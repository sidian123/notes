# 结构剖析

## manifest.json

> 必须有

包含了扩展的元数据, 如扩展名, 版本和权限等. 用到的其他文件也必须在此声明, 声明的文件有:

- [Background scripts](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Anatomy_of_a_WebExtension#Background_scripts): Implement long-running logic.
- Icons for the extension and any buttons it might define.
- [Sidebars, popups, and options pages](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Anatomy_of_a_WebExtension#Sidebars_popups_options_pages): HTML documents that provide content for various user interface components.
- [Content scripts](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Anatomy_of_a_WebExtension#Content_scripts): JavaScript included with your extension, that you will inject into web pages.
- [Web-accessible resources](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Anatomy_of_a_WebExtension#Web_accessible_resources): Make packaged content accessible to web pages and content scripts.

![img](.Firefox%20Extension/webextension-anatomy.png)

## Background scripts

> 后台脚本

### 介绍

* 用于执行长期运行的任务, 其声明周期独立于任何特定的网页和浏览器窗口.

* 扩展加载时,后台脚本被加载, 直到浏览器关闭或扩展被禁止, 卸载.

* 可以使用任何[WebExtension APIs](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/API), 只要有对应的权限.

### 使用

* 引入后台脚本

  ```json
  "background": {
    "scripts": ["background-script.js"]
  }
  ```

* 引入页面且使用ES6模块

  manifest.json

  ```json
  "background": {
    "page": "background-page.html"
  }
  ```

  background-page.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8">
      <script type="module" src="background-script.js"></script>
    </head>
  </html>
  ```

### 运行环境

* 后台脚本运行在特定的页面中, 称为**后台页面** ( Background Page ) , 因此可以使用所有DOM API.

  > Firefox中, 不支持`alert()`, `confirm()`, `prompt()`的使用
  >
  > 后台页面是不可视的.

* 可以使用任何 [WebExtension APIs](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/API), 只要有权限

* 对于有访问权限的主机,可以直接发起跨域XHR请求.

* 后台脚本不能直接访问网页, 但是可以通过[消息传送API](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_scripts#Communicating_with_background_scripts), 与内容脚本 ( content scripts ) 沟通, 从而操作网页.

* 由于安全限制, 一些危险操作不能使用, 如`eval()`, 见[Content Security Policy](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_Security_Policy)

## Sidebars, popups, options pages

扩展提供给用户的访问接口, 以HTML展示内容.

- a [sidebar](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/user_interface/Sidebars) is a pane that is displayed at the left-hand side of the browser window, next to the web page
- a [popup](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/user_interface/Popups) is a dialog that you can display when the user clicks on a [toolbar button](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/user_interface/Browser_action) or [address bar button](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/user_interface/Page_actions)
- an [options page](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/user_interface/Options_pages) is a page that's shown when the user accesses your add-on's preferences in the browser's native add-ons manager.

这些页面都属于**扩展页面** ( Extension pages ).

## 扩展页面

扩展页面用于提供额外的可视信息, 如下面的小窗口:

![Example of a simple bundled page displayed as a detached panel.](.Firefox%20Extension/bundled_page_as_panel_small.png)

> 扩展页面是没有地址栏, 书签栏等内容的

可由`windows.create()`和`tabs.create()`创建, 由`windows.remove()`删除.

扩展页面也能够访问WebExtension API, 可以通过[`runtime.getBackgroundPage()`](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/runtime/getBackgroundPage)与后台页面交流.

扩展页面包含Sidebars, Popups, Options pages, 就连后台页面, 也可当作隐藏的扩展页面.

> 参考[Extension pages](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/user_interface/Extension_pages)

## Content scripts

内容脚本 ( Content scripts ) 被加载到网页中, 用于访问和操作网页的DOM. 除此之外, 还能

* 发起跨域请求
* 使用部分WebExtension API
* 与后台脚本[通信](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_scripts#Communicating_with_background_scripts)
* 可通过[window.postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)与网页的脚本通信

## Web accessible resources

可让扩展的资源被内容脚本和页面脚本访问.

步骤如下:

1. 在manifest.json中注册这些资源
2. 通过`browser.extension.getURL()`获取资源URL

# WebExtensions API 

* 介绍
  * 可用于后台脚本和扩展页面中, 部分API也可用于内容脚本中.
  * 使用API前, 需在`manifest.json`中声明权限
  * 以`browser`为名字空间, 同时也支持`chrome`作为名字空间; 支持异步回调, 支持promises

* API 列表

  ...

> 参考[JavaScript APIs](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API)















# 参考

* [manifest.json](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json)