# 概要

* 介绍

  * 通用文本编辑器
  * 更多的用于编辑代码

* 组件

  * 编辑器: 由CodeMirror核心库提供.

  * 插件( addon ): 为编辑器提供更多的功能

    > CodeMirror提供了一些插件, 位于`addon/`目录下

  * Mode脚本: 为特定的语言提供高亮或缩进功能

    > 同样的, `mode/`目录下存在很多mode脚本
  
  * 主题: 覆盖核心库的样式, 提供其他的样式.
  
    > 位于`theme/`目录下

# 入门

* 安装

  ```shell
  npm install codemirror --save
  ```

* 引入: 至少引入`lib/`目录下核心库的Js和Css, 以及一个Mode脚本

  ```javascript
  import CodeMirror from "codemirror";
  import "codemirror/lib/codemirror.css"
  import "codemirror/mode/javascript/javascript.js"
  ```

* 生成CodeMirror实例

  ```javascript
  var myCodeMirror = CodeMirror.fromTextArea(myTextArea);
  ```

  > 第一个参数为`<textarea>`元素

# 配置

`fromTextArea()`的第二个参数可传入一个选项对象,  [CodeMirror.defaults](https://codemirror.net/doc/manual.html#defaults)属性提供默认选项.

如下列举常用配置: 

* ` value: string|CodeMirror.Doc `

  编辑器中的初始编辑内容

* ` mode: string|object `

  使用的Mode. 若无, 则使用第一个加载mode脚本.

  > 使用mode需要存在已加载的mode脚本, 脚本位于`mode/`目录下

* ` theme: string `

  主题, 默认`default`

  > 必须存在已加载的样式, 样式位于`theme/`目录下.

* ...

> 详细配置见[Configuration](https://codemirror.net/doc/manual.html#config)

# 事件

CodeMirror中存在多种对象可以发送事件

# Key Maps

用于关联按键与鼠标到功能的映射

# Commands

和编辑器相关的命令, 常用于映射到快捷键



# 自定义

## 自定义主题

设置了`theme`属性后, 如`theme:"hello"`, 那么CodeMirror的根元素将附有类`cm-s-hello`. 根据这个规则去覆盖CodeMirror原有样式即可.

# 其他

* CodeMirror只部分渲染在视窗内的内容, 因此效率较高.

# 参考

* [CodeMirror](https://codemirror.net/)