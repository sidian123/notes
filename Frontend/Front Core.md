* 为啥使用路由

  在使用MVVC框架后, 页面体积会很大, 加载也会很慢

  引入路由后, 页面跳转间, 之前加载的组件等资源, 可被重复利用.

* 开发模式

  以前是多页面、前后端不分离的方式开发，并且前端页面都是通过后端模板视图输出的。

  如今流行前后端分离开发，狭隘来说，前后端仅通过Json通信。更广泛的，开发时解决跨域问题，基本不需要前端的干涉，只需定义接口接口即可。开发完成后，前后端可整合，也可分离部署。

  并且逐渐流行单页面开发，但通过url上的hash也能完成一个含“多页面”的系统，即单页面中包含了“多页面”，通过webpack+vue可以轻松胜任。

  > 案例：[vuejs 和 element 搭建的一个后台管理界面](https://www.cnblogs.com/taylorchen/p/6083099.html)

* 拷贝到剪贴板

  [~~v-clipboard~~](https://www.npmjs.com/package/v-clipboard)

  [vue-clipboard2](https://www.npmjs.com/package/vue-clipboard2)

* 导出SVG到PNG

  [saveSvgAsPng](https://www.npmjs.com/package/save-svg-as-png)
  
* [Dom to Image](https://www.npmjs.com/package/dom-to-image)

  * 依赖

    ```shell
    npm i dom-to-image
    npm install file-saver
    ```

  * demo

    ```javascript
    domtoimage.toBlob(document.getElementById('my-node'))
        .then(function (blob) {
            window.saveAs(blob, 'my-node.png');
        });
    ```

    > 如果内容可滚动, 则转化显示的部分

  * demo2 转化内容包括scroll内容

    ```javascript
    domtoimage.toPng(node, { width: node.scrollWidth, height: node.scrollHeight })
    ```

    > 参考 https://github.com/tsayen/dom-to-image/issues/183#issuecomment-468606593

* SVG

  [Snap.svg](http://snapsvg.io/)

* JSX

  [JSX详解](https://www.jianshu.com/p/3345e94baec0)

* Vue生态

  [awesome-vue](https://github.com/vuejs/awesome-vue)

# lodash

> [lodash](https://lodash.com/)

* 安装

  ```shell
  npm i --save lodash
  ```

* 深拷贝

  ```js
  _.cloneDeep(value)
  ```

* 浅拷贝

  ```js
  _.clone(value)
  ```

* 防抖函数

  ```
  _.debounce(func, [wait=0], [options={}])
  ```

  * `debounce`返回一个防抖函数, 函数原型与`func`一致.
  * 多次调用防抖函数, `func`被执行当且仅当最后一次调用防抖函数与上次调用防抖函数的时间超过了`wait`
  * `func`默认在`wait`结束后执行, 也即`options.trailing=true`; 可设置`options.leading=true`, 让`func`在`wait`开始前执行
  * 防抖函数可能会一直处于等待状态, `options.maxWait`可设置`func`被执行的最大等待时间.

  > 要理解上述内容, 需要区分**防抖函数**与`func`

# 时间

## date-format

> [date-format](https://www.npmjs.com/package/date-format)

安装

```shell
npm install date-format
```

使用

```javascript
  /**
   * 日期格式化
   * @param date 时间戳
   * @returns {string} yyyy-MM-dd hh:mm:ss类型的日期格式
   */
  dateFormat(date){
    return date==null || date === 0 ? '' : dateFormat('yyyy-MM-dd hh:mm:ss', new Date(date))
  }
```

## Moment

> [Moment.js](http://momentjs.cn/)

安装

```bash
npm install moment --save
```

使用

```js
moment().format("YYYY-MM-DD HH:mm:SS");
```

