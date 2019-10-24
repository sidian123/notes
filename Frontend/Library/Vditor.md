# 入门

* 安装

  ```shell
  npm install vditor --save
  ```

* 引入

  ```javascript
  import Vditor from 'vditor'
  import "vditor/src/assets/scss/classic" // 或者使用 dark
  
  const vditor = new Vditor(id, {options...})
  vditor.focus()
  ```

* 使用

  ```html
  <template>
      <div>
        <div id="editor"></div>
      </div>
  </template>
  <script>
      import Vditor from "vditor"
      import "vditor/src/assets/scss/classic.scss"
      export default {
          name: "Test",
          mounted(){
              const vditor = new Vditor("editor");
              vditor.focus()
          }
      }
  </script>
  ```

* 使用形式

  ```javascript
  const vditor = new Vditor(id, {options...})
  ```

# 选项

记录部分选项

* `options`基本设置
  * `width`: 宽度, 默认`auto`, 支持`%`
  * `height`: 高度, 默认`auto`
  * `tab`: tab键代表的字符, 默认无
  * `typewriterMode`预览视图是否跟随编辑视图滚动, 默认`false`
* `options.toolbar`自定义工具类, 已提供的工具可用`name`简写声明, 未提供的需自己提供所有选项.
* `options.preview`: 显示相关
  * `mode`: 显示模式, 如`both`(双栏), `editor`(仅编辑), `preview`(仅预览)
* `options.preview.hljs`: 与预览样式相关
* `options.hint`: 与表情相关
* `options.upload`: 与图片上传有关
* `options.resize`: 与组件拖拽有关
* `options.classes`: 配置预览元素的class名

# 方法

记录部分方法

## 实例方法

| 实例方式                                        | 说明                                            |
| ----------------------------------------------- | ----------------------------------------------- |
| getValue()                                      | 获取编辑器内容                                  |
| getHTML()                                       | 获取预览区内容。该方法需使用异步编程            |
| insertValue(value: string)                      | 在焦点处插入内容                                |
| focus()                                         | 聚焦到编辑器                                    |
| blur()                                          | 让编辑器失焦                                    |
| disabled()                                      | 禁用编辑器                                      |
| enable()                                        | 解除编辑器禁用                                  |
| setSelection(start: number, end: number)        | 选中从 start 开始到 end 结束的字符串            |
| getSelection():string                           | 返回选中的字符串                                |
| setValue(value: string)                         | 设置编辑器内容                                  |
| renderPreview(value?: string)                   | 设置预览区域内容                                |
| getCursorPosition():{top: number, left: number} | 获取焦点位置                                    |
| deleteValue()                                   | 删除选中内容                                    |
| updateValue(value: string)                      | 更新选中内容                                    |
| isUploading()                                   | 上传是否还在进行中                              |
| clearCache()                                    | 清除缓存                                        |
| disabledCache()                                 | 禁用缓存                                        |
| enableCache()                                   | 启用缓存                                        |
| html2md(value: string)                          | HTML 转 md。该方法需使用异步编程                |
| tip(text:string, time:number)                   | 消息提示。time 为 0 将一直显示                  |
| setPreviewMode(mode: string)                    | 设置预览模式。mode: 'both', 'editor', 'preview' |

## 静态方法

如果不需要编辑操作时, 可仅引入部分库:

```javascript
import VditorPreview from 'vditor/dist/method.min'
```

| 静态方法                                                     | 说明                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| mathRender(element: HTMLElement)                             | 转换 element 中的文本为数学公式                |
| mermaidRender(element: HTMLElement)                          | 转换 element 中的文本为流程图/时序图/甘特图    |
| codeRender(element: HTMLElement, lang: (keyof II18nLang) = "zh_CN") | 为 element 中的代码块添加复制按钮              |
| chartRender(element: (HTMLElement                            | Document) = document)                          |
| abcRender(element: (HTMLElement                              | Document) = document)                          |
| md2html(mdText: string, options?: IPreviewOptions): string   | markdown 文本转换为 HTML，该方法需使用异步编程 |
| preview(element: HTMLTextAreaElement, options?: IPreviewOptions) | 页面 Markdown 文章渲染                         |
| highlightRender(hljsStyle: string, enableHighlight: boolean, element?: HTMLElement | Document)                                      |
| mediaRender(element: HTMLElement)                            | 为特定链接分别渲染为视频、音频、嵌入的 iframe  |
| mathRenderByLute(element: HTMLElement)                       | 对使用 Lute 渲染的数学公式进行渲染             |

其中, `IPreviewOptions`对象原型如下

```javascript
{  
 hljsStyle?: string;    // 高亮样式，默认为 'atom-one-light'  
 enableHighlight?: boolean;    // 是否需要代码高亮，默认为 true  
 customEmoji?: { [key: string]: string };    // 自定义 emoji，默认为 {}  
 lang?: (keyof II18nLang);    // 语言，默认为 'zh_CN'  
 emojiPath?: string;    // 表情图片路径  
}  
```

# 使用

用法见 [Blog的组件代码](https://github.com/sidian123/Blog/tree/master/front/components/editor) 

但有几点需要注意下:

* 不用使用`textarea`元素作为Vditor的容器, 即`id`的元素最好为`div`

# 参考

> 参考[一款浏览器端的 Markdown 编辑器](https://hacpai.com/article/1549638745630)