# 一 基础

## 浏览器解析过程

浏览器有三个解析器：html parser，javascript parser 和 css parser。

html解析器最先开始运行，可以调用其他两个解析器。**从上到下按照顺序解析每个元素标记**，生成元素、注册到DOM名字空间中、属性添加到dom元素节点上。

如果遇到css，则**加载**并调用css解析器**解析**。此过程与html解析器同时进行，非阻塞，因为css规则会一直被**应用**。这是为什么新定义的css规定能够导致元素重画的原因。

如果遇到javascrpt，则**加载**并调用javascript解析器**运行**。会导致html解析器停止运行、阻塞，直到javascript运行完毕。

参考：<https://stackoverflow.com/questions/1795438/load-and-execution-sequence-of-a-web-page>

## 引入javascript

在html中引入javascript中有三种方式：

1. internal javascript

   ```
   <script>
     // JavaScript goes here
   </script>
   ```

2. external javascript

   ```
   <script src="script.js"></script>
   ```

3. inline javascript handles

   ```
   <button onclick="createParagraph()">Click me!</button>
   ```

## javascript加载问题

**问题**：

1. javascript在加载并解析时会导致html解析器停止运行，导致网页的加载性能下降。
2. javascript在加载并运行时，html元素并未全部被解析，因此js操作元素会出错。

**办法**：

1. `script`元素放到网页最底部,仅仅`</bdy>`标签前面。解决第一个问题。
2. 脚本中监听window的onload事件，文档全部加载完毕才执行js。解决第二个问题。
3. 使用script的async或defer属性。async异步加载，但是加载完后执行时还是会阻塞html解析器，能解决第一个问题；defer异步加载，但是在**html解析完毕后、document的onload事件出发前**执行js，同时能直接两个问题。

**注意**：有async属性的script只能是外部链接，而且执行顺序不确定；而defer**执行顺序与源码一致**。由于defer脚本是在onload事件前执行的，因此此时css和图片可能仍在加载与解析。

参考：<https://flaviocopes.com/javascript-async-defer/>

# 二 DOM

## 介绍

DOM（Document Object Model）是一个用于操作HTML或XML文档的编程**接口**。DOM将文档的内容当作**节点以树的方式**表示文档。并且是与语言无关的，DOM只定义了每个接口的内容。

DOM接口大致分为两类：DOM核心接口和与html元素相关的接口。DOM核心接口定义了一些主要和DOM文档操作、事件相关的接口。其中`Node`接口代表节点，`Element`接口继承Node，代表元素；而其他接口定义了和html元素相关的接口，含有对应元素特定的属性。其中`HTMLELement`代表html元素，然后是继承该接口的其他更具体的接口，比如`HTMLTableElement`。

在实际使用中，都是对对象操作的，那对象与接口的关系如何？接口与接口之间是有继承关系的，因此对象可以使用多个接口定义的属性或方法。比如`<p>`对应的接口`HTMLParagraphElement`，该接口可以设置和p元素相关的属性；它的继承对象为`HTMLElement`，该接口拥有操作html元素通用的方法；在之上为`Element`、`Node`，提供了操作dom节点的方法。经常使用的还是window和document对象，前者代表浏览器，后者代表文档，提供了查找某个元素的方法。
![在这里插入图片描述](.Web%20APIs/20181204202507391.png)

## 常用操作

* 查找相关
  * Document.querySelector()：通过css选择器来选择html元素。
  * Document.querySelectorAll()
  * Document.getElementById()
  * Document.getElementsByTagName()

-----------

* DOM树操作相关
  * Document.createElement()
  * Node.appendChild()
  * Node.removeChild()
  * ` Element.remove()`删除自身
  * `Node.contains()` 判断子节点或当前节点是否包含参数代表的节点.

-----------

* 节点内容相关

  * Element.innerHTML

  * `HTMLElement.innerText`

  * `Node.textContent`

    > `innerXXX`与`textContent`相比
    >
    > * `innerXXX`会额外将样式考虑进去, 区别见[textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)
    > * `innerHTML`会decode HTML实体, 如 `&gt;` => `>` ; 而`textContent`会encode 保留字符, 如 `>` => `&gt;`

  * element.style.left

  * element.setAttribute()

  * element.getAttribute()

  * element.addEventListener()

----------

* 其他

  * window.content

  * window.onload

  * console.log()

  * `Element.scrollTo()` 滚动元素内容, 如平滑滚动

    ```javascript
    element.scrollTo({
      top: 100,
      left: 100,
      behavior: 'smooth'
    });
    ```

> 参考：[Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)

## Window

Window接口表示一个包含了DOM文档的窗口，在浏览器中具体表现为一个标签页。Window接口的对象window含有所有和窗口相关的内容，也是JavaScript的运行环境，JavaScript中的全局变量、函数就是该window对象的属性或方法；除了DOM和脚本外，还涉及其他对象，如location（代表链接）、History（代表理事）等等。

* 获取窗体大小
  * ` window.innerWidth `窗体内容宽度
  * `window.innerHeight`窗体内容高度
  * `window.outerWidth`窗体总的宽度,包含工具栏,滚动条
  * `window.outerHeight`窗体总的高度,包含工具栏,滚动条
* 事件
  *  [resize](https://developer.mozilla.org/en-US/docs/Web/API/Window/resize_event) 窗体大小改变时触发

* 滚动
  * `Window.scrollTo()` 滚动条来自window时可用

>具体使用参考
>
>* [Window MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window)
>* [Wndow w3schools](https://www.w3schools.com/jsref/obj_window.asp)

## Document

* 介绍

  代表整个文档

* 操作
  * 查询元素
  * 很多文档相关的全局事件从中发出
  * ...

* 事件

  * `visibilitychange` 当标签页可视或隐藏时触发

    ```javascript
    document.addEventListener("visibilitychange", function() {
      if (document.visibilityState === 'visible') {
        backgroundMusic.play();
      } else {
        backgroundMusic.pause();
      }
    });
    ```

> 参考[Document](https://developer.mozilla.org/en-US/docs/Web/API/Document)

## Location

* 其对象`location`是`window`对象的属性, 代表当前url
* 提供了一些属性用以访问url
* 提供了跳转的方法
  * `reload()`重新加载
  * `replace()`替换当前页面, 无历史记录
  * `assign()`加载新页面, 有历史记录, 即可以会退到上一页面.

> 参考[Location mdn](https://developer.mozilla.org/en-US/docs/Web/API/Location)

## History

代表浏览器的会话历史, 通过该接口, 可以后退前进来浏览历史, 和添加或替换新历史记录.

> 注意, **修改历史**记录不会造成浏览器重新加载. 但**浏览历史**会造成页面重载, 但存在特例, 见下.
>
> 更改`Location.hash`也可达到创建新历史记录而不加载的目的, 但是限制较多.略

**浏览历史**的三个方法:

* [`back()`](https://developer.mozilla.org/en-US/docs/Web/API/History/back)后退
* [`forward()`](https://developer.mozilla.org/en-US/docs/Web/API/History/forward)前进
* [`go()`](https://developer.mozilla.org/en-US/docs/Web/API/History/go)从指定历史记录中加载页面

**修改历史**:

* [`pushState(state,title,url)`](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState)添加历史记录
  * `state`关联到历史记录的一个对象, 存数据用的. 可传空对象`{}`
  * `title`标题. 可传空字符串`""`
  * `URL`新历史记录的url,不会被加载, 可以为相对地址, 如`?page=1`, `#a23`, `index.html`等等
* [`replaceState(state,title,url)`](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState)替换历史记录, 参数同上

------

**可能会遇到的bug:**

通过`pushState`设置历史记录, 如果设置后仍处于同一页面, 那么`back()`或点击浏览器按钮, 仅修改历史, 不重载, 但会发送[popstate](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)事件, 通过[window.onpopstate](https://developer.mozilla.org/en-US/docs/Web/API/WindowEventHandlers/onpopstate)捕获

## Screen

`screen`对象含有与访问者屏幕相关的信息, 如

| Property                                                     | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [availHeight](https://www.w3schools.com/jsref/prop_screen_availheight.asp) | Returns the height of the screen (excluding the Windows Taskbar) |
| [availWidth](https://www.w3schools.com/jsref/prop_screen_availwidth.asp) | Returns the width of the screen (excluding the Windows Taskbar) |
| [colorDepth](https://www.w3schools.com/jsref/prop_screen_colordepth.asp) | Returns the bit depth of the color palette for displaying images |
| [height](https://www.w3schools.com/jsref/prop_screen_height.asp) | Returns the total height of the screen                       |
| [pixelDepth](https://www.w3schools.com/jsref/prop_screen_pixeldepth.asp) | Returns the color resolution (in bits per pixel) of the screen |
| [width](https://www.w3schools.com/jsref/prop_screen_width.asp) | Returns the total width of the screen                        |

> 貌似不准确, 请用`window.innerWidth`和`window.innerHeight`

# 三 事件

## 介绍

**事件**是系统（browser）中某个动作或事件的发生，系统会产生一个信号（**event**），这个信号含有关于该事件的信息。随着信号的产生，**事件处理器或监听器**会采取一定动作处理该事件。这种事件处理机制称为**事件模型**。事件模型不是javascript特有的，不同的上下文中会有不同的**事件模型**，即事件模型的实现机制不太一样，比如web api、插件、Node.js等等上下文。

这里讲的事件模型是DOM相关的，所有的事件都在DOM中发生，比如用户产生的（鼠标、键盘事件），或者被API产生（动画结束、视频暂停等）。也能够通过代码触发事件，如`HTMLElement.click()`模拟点击事件；或者自定义事件，通过`EventTarget.dispatchEvent()`发送给某个元素。`Event`是最泛化的事件接口，其他接口都继承该接口。

许多DOM元素都可以监听事件，然后执行元素的事件处理器。通过`EventTarget.addEventListener()`可以让事件处理器附着在元素上，并且同一事件还可以同时被多个处理器监听。`removeEventListener()`则可以删除连接的事件。

事件传播一般是从DOM树中从下至上传播的，即**bubbling**。由于同一事件可能有多个处理器，因此事件被传到一个元素时，会横向传播，然后在向上传播（bubbling）。一些元素都有对事件有默认行为，如链接点击会打开网页、表单按钮点击发送表单等等，默认行为会在事件横向传播时执行，通过`Event.preventDefault()`可以阻止默认行为。`Event.stopPropagation()`可以阻止事件向上传播，但允许横向传播。`Event.stopImmediatePropagation()`同时阻止事件横向和向上传播。

> 参考
>
> * [Event](https://developer.mozilla.org/en-US/docs/Web/API/Event)
>
> * [全局事件处理器](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers)
> * [所有事件](https://developer.mozilla.org/en-US/docs/Web/Events)
> * [event接口](https://developer.mozilla.org/en-US/docs/Web/API/Event)
> * [事件的所有键值key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)

## 基础

### 监听方案

一共有三种方法可以使用事件：

1. **Event handler properties**：html元素对应的javascript对象存在一些用于执行事件处理器的属性。

   ```javascript
   var btn = document.querySelector('button');
   btn.onclick = function() {...};
   ```

2. **Inline event handlers**：不建议使用。html元素的一些属性可以指定事件处理的javascript**代码块**。

   ```javascript
   <button onclick="alert('Hello, this is my old-fashioned event handler!');">Press me</button>
   ```

3. **addEventListener() and removeEventListener()**：前者可以为事件注册事件监听器**且同时能注册多个**，后者能够删除事件监听器。

   ```javascript
   //add handler
   btn.addEventListener('click', bgChange);
   //remove handler
   btn.removeEventListener('click', bgChange);
   //add multiple handlers
   myElement.addEventListener('click', functionA);
   myElement.addEventListener('click', functionB);
   
   ```

> 如果2,3都存在时, 先执行2

> 注意! 监听器中的`this`指向监听的元素; 但如果传入的是Vue实例的方法, 则`this`仍指向Vue实例

### Event object

事件发生后，会同时产生event对象，通过该对象可以获得关于事件的信息，比如Event.target指定具体产生event的对象。

事件处理器都可以接收event对象，只要函数定义时指定参数即可：

```javascript
let button=document.querySelector('div');
button.addEventListener('click',function(event){
  console.log(event.target);
});
```

`Event`对象是一个泛型类，含有事件通用的方法和属性。每个具体的事件还有它特有的属性方法。

下面是`Event`的有用的属性和方法：

1. `event.target` – the deepest element that originated the event.
2. `event.currentTarget (=this)` – the current element that handles the event (the one that has the handler on it)
3. `event.eventPhase` – the current phase (capturing=1, bubbling=3).
4. `event.stopPropagation()`：事件停止向上传播，但在当前元素内可以纵向传播给同一事件的其他处理器。
5. `event.stopImmediatePropagation()`：停止事件横向、纵向传播。
6. `event.defaultPrevented()`：阻止默认行为。注意，在html元素的事件处理属性中`return false`也能阻止默认行为，但是在其他地方行不通，因此不建议使用。

### Event bubbling and capture

当事件发生时，event对象会传播给目标对象，执行处理器。但是这个过程分为三个阶段：

1. Capturing phase – the event goes down to the element.
2. Target phase – the event reached the target element.
3. Bubbling phase – the event bubbles up from the element.

默认处理器时注册在bubbling阶段的，因此先执行target对象的处理器，然后执行顶层元素的处理器。

addEventListener的第三个参数为true时（默认false）注册在capturing阶段。

target阶段不能单独存在，而是包含在其他两个阶段内。比如，capturing阶段最后一个处理器和budding阶段第一个处理器就发生在该阶段。

### 事件代理

这里的代理就是指父类来处理子类的事件。比如点击`li`，但监听器设置在`ul`上，通过event.target就知道具体哪个元素被点击，于是可以方便其代为处理该事件。利用事件传播原理。

参考：<https://javascript.info/bubbling-and-capturing>

## 事件使用

### 键盘事件

> 一般表单元素可接收键盘事件, 若让其他元素也接收键盘事件, 可使用属性`tabindex='-1'`

有三种与键盘相关的事件

* [keydown](https://developer.mozilla.org/en-US/docs/Web/Events/keydown)按下键时触发
* [keypress](https://developer.mozilla.org/en-US/docs/Web/Events/keypress)键值可打印时触发
* [keyup](https://developer.mozilla.org/en-US/docs/Web/Events/keyup)松开按键时触发

事件触发后, 都由[KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)对象表示.

触发顺序也同上. 如果长按时, 其事件触发顺序为:

`keydown`->`keypress`->`keydown`->`keypress`->`keydown`->...->`keyup`

`keyboardEvent`对象有用的属性如下:

* `ctrlKey`是否按下control键
* `altKey`是否按下alt键
* `shiftKey`是否按下shift键
* `code`按键完整的名字, 如`ControlLeft`,`KeyA`
* `key`按键的简称, 如`Control`,`a`

> 参考:[KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)

### 触碰事件

在触碰敏感的平面上, 触碰状态改变时, 会触发触碰事件[TouchEvent](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent) .每个触碰点被描述为[Touch](https://developer.mozilla.org/en-US/docs/Web/API/Touch)对象,含有位置,大小,形状,压力等信息.

> 该平面可以是触摸屏或触摸板

触碰事件分为四种:

* `touchstart`用户触碰时触发
* `touchend`离开触碰面时触发
* `touchmove`移动触碰点时触发
* `touchcancel`触碰点被破坏??? 略

> 好像电脑上触发不了该事件

事件上有用的属性:

* `targetTouches`: 同时被触碰且触碰到同一个元素上的`Touch`对象集合

* [`TouchEvent.changedTouches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/changedTouches)  

* [`TouchEvent.touches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/touches)  

  > 有点分不清了...

> `Touch`上可通过`clientX`,`clientY`获取其位置

### 鼠标事件

与触碰类似, 不过触碰不能用于PC端, 而鼠标事件只能用于PC端

常用的鼠标事件有:

* `mousedown`鼠标在元素上按下时触发

* `mousemove`鼠标在元素上移动时触发

  > 貌似`mousemove`不灵敏, 即事件触发频率不高

* `mouseup`鼠标在元素上抬起时触发

* `click`在`mousedown`和`mouseup`事件触发后触发, 即点击了元素.

> 参考[MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)

### 拖拽事件

* 介绍

  `DragEvent`接口代表拖拽事件.

* 具体事件
  * `drag `拖动一个可拖拽元素时触发
  * `drop` 在可放置区域放下时
  * ~~`dragend` 无论是否拖拽成功, 拖拽结束后触发~~
* ...
  
* `DataTransfer`

  拖拽事件的数据通过其属性`dataTransfer`来传递.

  * `items` 拖拽事件传递的数据集合.
  * `types` 数据的类型集合, 与`items`位置一一对应.
  * `setData()` 添加指定类型的数据. 若`items`中不存在该类型, 则添加, 否则替换.
  * `getData()` 获取指定类型的数据
  * `clearData()` 清楚指定类型的数据
  * ...
  
* 使用步骤

  1. 拖拽的对象必须是可拖拽的, 需声明`draggable`属性

      ```html
      <p id="p1" draggable="true">This element is draggable.</p>
      ```

  2. `dragstart`事件上定义拖拽的数据
  
     ```javascript
     function dragstart_handler(ev) {
       // Add different types of drag data
       ev.dataTransfer.setData("text/plain", ev.target.innerText);
       ev.dataTransfer.setData("text/html", ev.target.outerHTML);
       ev.dataTransfer.setData("text/uri-list",ev.target.ownerDocument.location.href);
     }
     ```
  
  3. 定义拖拽效果, 会影响拖拽过程中光标的样式, 可选值`copy|move|link`等
  
     ```java
     function dragover_handler(ev) {
       ev.dataTransfer.dropEffect = "copy";
     }
     ```
  
     > 仅在`dragover`事件中设置即可
  
  4. 定义可放置区域, 同时注册`dragover`,`ondrop`事件处理器, 并调用`preventDefault()`即可
  
     ```html
     <script>
     function dragover_handler(ev) {
      ev.preventDefault();
      ev.dataTransfer.dropEffect = "move";
     }
     function drop_handler(ev) {
      ev.preventDefault();
      // Get the id of the target and add the moved element to the target's DOM
      const data = ev.dataTransfer.getData("text/plain");
      ev.target.appendChild(document.getElementById(data));
     }
     </script>
     
     <p id="target" ondrop="drop_handler(event)" ondragover="dragover_handler(event)">Drop Zone</p>
     ```

> 参考
>
> * [HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)
> * [DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)

### 剪贴板事件

* 介绍

  `ClipboardEvent`接口代表剪贴板事件, 含有`clipboardData`属性, 能够获取和设置剪贴板内容. 
  
* 事件

  > 以下都是`ClipboardEvent`的实例对象

  * `paste`粘贴事件

    当`paste`事件发生且光标处于可编辑元素中时, 默认行为是将剪贴板中内容插入到文档中( 会带有格式 ).

  * `cut`

  * `copy`

* 自定义行为

  `ClipboardEvent.clipboardData` 是一个`DataTransfer`对象, 剪贴板内容将存入`items`中. 

  `cut`, `copy`事件默认行为都调用`DataTransfer.setData()`方法, `paste`事件默认行为调用了`DataTransfer.getData()`方法

  > 具体使用见 *拖拽事件*

> 参考
>
> * [ClipboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent)
> * [DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)

### 范围选择事件

* 事件

  * `selectstart` 
  * `selectionchange`

  > 见[Selection API](https://w3c.github.io/selection-api/)

* 阻止

  Chrome和Firefox的范围选择功能不一致, 统一阻止范围选择的方案如下:

  ```javascript
  function preventSelection(event){
      event.preventDefault();
  }
  element.addEventListener("mousedown",ev->{
  	document.addEventListener("selectstart",preventSelection)
  	event.preventDefault();
  })
  element.addEventlistener("mouseup",preventSelection)
  ```

  > 见[javascript: Disable Text Select](https://stackoverflow.com/a/27434703)

### 其他事件

* [scroll](https://developer.mozilla.org/en-US/docs/Web/API/Document/scroll_event) 滚动事件, 当元素滚动时触发

# 四 API

##  存储

html5后引入了web storage（本地储存），比cookies更好用。本地存储分两类：

* `window.localStorage` - stores data with no expiration date
* `window.sessionStorage` - stores data for one session (data is lost when the browser tab is closed)

使用`setItem`保存键值对，`getItem`获得键值对。注意，键值对被存为字符串形式，因此存入和取出时做好转化。

>参考：[HTML5 Web Storage](https://www.w3schools.com/html/html5_webstorage.asp)

## 编,解码

### URL

* `encodeURI()`, `decodeURI()`编码或解码URL

* `encodeURIComponent()`, `decodeURIComponent()` 编码或解码URL组件, 即`(; / ? : @ & = + $ , #)`等字符间的内容.

  > 因此`encodeURIComponent()`传入`/`会被编码的

### Base64

* `atob()`解码
* `btoa()`编码

### JSON

* `JSON.parse()`解码为JavaScript对象

  > 也可用于判断Json语法是否正确

* `JSON.stringify()`编码为Json字符串

  编码的同时格式化

  ```javascript
  JSON.stringify(value,null, space)
  ```

  > `space`表示缩进

> 参考
>
> * [Base64 encoding and decoding](https://developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding)
> * [JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON)

## 文件相关

### Blob

* 介绍

  表示二进制数据.

* 属性

  * `size` 大小, 单位字节
  * `type` MIME类型

* 方法 (不常用)

  * `arrayBuffer()` 解析为二进制数组缓存
  * `text()` 以UTF-8编码解析二进制, 得到文本.

> 参考[Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob)

### File

* 介绍

  `File`继承于`Blob`, 表示本地文件;

  网页环境中仅可以通过`<input type="file">`元素和拖拽文件获取.

  > 在其他特权环境中, 如插件, 可通过文件名直接获取, 见[Using the DOM File API in chrome code](https://developer.mozilla.org/en/Extensions/Using_the_DOM_File_API_in_chrome_code)

* 获取`File`对象

  * 通过`input`元素的`files`属性, 这是一个数组.

    ```html
    <input type="file" id="input">
    ```

    ```javascript
    const selectedFile = document.getElementById('input').files[0];
    ```

    > `input`元素需要加上`multiple`属性才能选择多个文件

    > 可以将获取逻辑写在`change`事件上.

  * 通过拖拽事件获取

    获取该事件对象的`dataTransfer`的`files`属性, 这是一个数组.

    ```javascript
    dropEvent.dataTransfer.files
    ```

* `File`对象常用属性

  * `name` 文件名, 不包括路径
  * `size` 文件大小, 单位字节
  * `type` 文件的MIME类型.

* 从`input`元素上获取`File`对象的两种方式

  * 隐藏`input`元素, 通过`click()`方法模拟点击

    ```html
    <input type="file" id="element" multiple accept="image/*" style="display:none" onchange="handleFiles(this.files)">
    <button id="button">Select some files</button>
    ```

    ```javascript
    document
        .getElementById("button")
    	.addEventListener("click", function (e) {
    		document.getElementByid("element").click();
        });
    ```

  * 使用`label`触发`input`元素

    `label`的特点: 点击`label`相当于点击`input`. 然后隐藏`input`元素, 这里有个注意事项, 略...

> 参考
>
> * [File](https://developer.mozilla.org/en-US/docs/Web/API/File)
> * [Using files from web applications](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications)
> * [Using the DOM File API in chrome code](https://developer.mozilla.org/en/Extensions/Using_the_DOM_File_API_in_chrome_code)
> * [DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)

### FileReader

* 介绍

  用于**异步**读取`Blob`对象内容

* 部分方法

  * `readAsArrayBuffer()`
  * `readAsBinaryString()`
  * `readAsDataURL()` 读取为`data:`协议的URL. 数据以base64编码的字符串呈现.
  * `readAsText()` 以指定编码解析为字符串
  
  > 注意, 读取完毕后, 会触发`load`事件, 通过该事件可以获取数据. 方法本身不立刻返回数据.

> 参考
>
> * [FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)
> * [Data URLs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)

## URL

* 介绍

  通过`URL`接口, 可以方便的操作URL链接.

* 对象URL

  * `createObjectURL()` 读取Blob为`blob:`协议的URL. 数据格式倒是不晓滴.

  * `createObjectURL()` 释放URL

    > 每次调用`createObjectURL()`后必须调用`createObjectURL()`, 否则占内存. 内部逻辑不晓滴

> 参考[URL](https://developer.mozilla.org/en-US/docs/Web/API/URL)

## blob&URL

* `blob:`与`data:`的区别

  不晓滴, 只知道`FileReader.readAsDataURL()`创建的URL不用释放?

* 从Object URL中获取Blob

  见[How to get a file or blob from an object URL?](https://stackoverflow.com/questions/11876175/how-to-get-a-file-or-blob-from-an-object-url)

# 其他

## 元素大小与位置

主要为了理清各种大小和位置属性之间的关系, 下面直接看三幅图

![Vertical sizing and positioning values for a child element](.Web%20APIs/hh781509.ie9_positioning1(en-us,vs.85).png)

> 红色为一个元素, 蓝色是它的父元素, 处于视窗内. 其中红色元素有滚动条

一些属性的解释:

* 元素自身模型大小

  * `getComputedStyle().height`元素内容的高

  * `getComputedStyle().paddingTop`元素的padding高

  * `getComputedStyle().borderTopWidth`或`clientTop`元素的border高

  * `getComputedStyle().marginTop`元素的边距高

    -------

  * `clientHeight`元素的高, 包括`padding`

  * `offsetHight`或`getBoundingClientRect().height`元素的高, 包括`border`

  * `scrollHeight`元素的高, 包括未显示的所有内容高度.

  > 注意, `getComputedStyle()`是`window`的方法

* 元素位置相关

  * `scrollTop`元素已**滚动**内容的距离

    > 注意, 该内容可滚动

    >若仅`window`可滚动, 请用`window.scrollTo()`

* `offsetTop`元素相对于**最近positional父元素**的距离

  >最好给父元素添加个`position`css属性

    > 因此, 要滚动元素内容时, 可让父元素的`scroll Top`等于滚动到的元素的`offsetTop`值.

  * `getBoundingClientRect().top`元素上方`border`边缘距离视窗上方边缘的距离.

  * ...

* ...

其他方向的属性类似, 略

--------

接着看下一幅图

![Vertical sizing and mouse coordinate positions affected by CSS transforms](.Web%20APIs/hh781509.ie9_positioning2(en-us,vs.85).png)

主要看`layerY`和`offsetY`的区别:

* 两者都时与鼠标事件相关的属性

* css transforms作用时, `offsetY`的坐标轴会跟着改变, 而`layerY`则不会
* `offsetY`不考虑`border`, `layerY`包含了`border`大小

-----------

![All vertical mouse coordinates and viewport offsets on an untransformed element](.Web%20APIs/hh781509.ie9_positioning3(en-us,vs.85).png)

这里也是和鼠标事件相关的属性:

* `offsetY`离元素padding边缘的距离
* `layerY`离元素border边缘的距离
* `clienty`离视窗边缘的距离
* `pageY`离整个页面的距离
* `window.pageYOffset`视窗离页面的距离

> 参考[Measuring Element Dimension and Location with CSSOM in Windows Internet Explorer 9](https://docs.microsoft.com/en-us/previous-versions//hh781509(v=vs.85)?redirectedfrom=MSDN&Accept-Language=zh-cn)

## DevTools

* Chrome的调试器

* **元素获取**: 在Chrome控制台中,`$()`是`document.querySeletor()`的别名, `$$()`是`document.querySelectorAll()`的别名

  > 千万不要用这个赋值,我发现是个大坑
  >
  > 好像可直接输入ID, 直接获取DOM元素

* **搜索文件**`Ctrl+P`

# 参考

* [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API)
* [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)

