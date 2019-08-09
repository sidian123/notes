[TOC]

# 一 介绍
与JQuery相比，Vue可以方便的进行组件开发。Vue组件相当于扩展的html元素，使用十分方便。

* vue能够操作文本、属性和结果，也提供了在元素被vue插入、更新、删除时的过渡效果(transition effects)
* 在vue中，我们并不直接操纵DOM，而是修改vue实例数据，让vue来修改dom。
* 模板中的数据绑定，一般可以绑定vue实例的大部分属性，只要这个属性不太复杂，vue会使用`stringify()`将之转化为string
* vue中重要关键词：数据绑定、虚拟DOM、响应式系统、响应式对象、DOM template、String template
* 虚拟DOM在应用到DOM时，会使用patch/reuse的算法减少DOM的操作。这种行为在你的输出结果不考虑子组件或DOM状态（如input值）时最合适。使用`key`属性指定唯一值可以取消这种行为。
* template和对应的html都存在时，template优先级高。
* 组件必须只存在一个根元素
* `v-bind`不使用指令参数，且值为对象，会将对象属性绑定到元素（组件上）
# 二 指令(directive)
vue指令以`v-`开始，用于渲染DOM或绑定数据，定义在元素或组件上。
* `{{message}}`：绑定vue实例数据`data`到文本上。（非指令，占位符）
* `v-bind`：将vue实例数据绑定到元素属性上。
* `v-if`：vue实例数据为`true`则显示该元素。
* `v-for`：遍历数组，产生多个该属性对应的html元素（及其子元素）。
* `v-on`：绑定vue实例方法到html元素事件上。
* `v-model`：实现input的value属性与vue实例数据的双向绑定，即value属性值改变也会影响vue实例数据。使用`v-bind`只能单向绑定inptu的value属性。

指令详细见第四章。注意，指令的值一般可以是表达式。
# 三 组件
* **组件本质上是一个vue实例，含有预定义好的选项**。但有些选项是特定于根组件的,如（`el`），根组件只能使用`new`创建。

## 3.1 组件实例
* 组件实例化有三种方式：
    * **html元素**：在html，则将组件名当作新的html标签，然后创建该元素。但这种实例方式不能作为根组件，需要放入额外一个Vue实例下。
	    >注意，组件作为`<ul>`的子元素实例化会出现问题，因此需使用`is`，如：
	    
	    ```html
	     <ul>
		    <li
		      is="todo-item"
		      v-for="(todo, index) in todos"
		      v-bind:key="todo.id"
		      v-bind:title="todo.title"
		      v-on:remove="todos.splice(index, 1)"
		    ></li>
	  </ul>
		```
		>`is`指定组件，实例化后该组件会取代`li`元素。
		
    * **Vue实例**：在javascript中，与`new Vue({...})`类似，`new`一个组件，并传入选项对象，绑定到一个html元素上(使用`el`选项)。此时被绑定的元素会被模板`template`替代。
    * **Vue子类实例**：Vue实例只能用于根组件，而Vue子类实例可以用于其他组件。`Vue.extend()`中传入选项对象，返回一个Vue子类。然后实例化，并用`$mount`挂载到DOM上。如下所示：
    	```html
    	<div id="mount-point"></div>
    	```
    	```css
    	// create constructor
		var Profile = Vue.extend({
		  template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
		  data: function () {
		    return {
		      firstName: 'Walter',
		      lastName: 'White',
		      alias: 'Heisenberg'
		    }
		  }
		})
		// create an instance of Profile and mount it on an element
		new Profile().$mount('#mount-point')
    	```
    	>注意data为函数。
    	>如果想手动销毁实例，则使用`$destroy`生命周期方法，但DOM元素仍未去除。
* 定义组件：
    ```javascript
    //定义一个叫todo-item的组件，成为新的html元素。还传入了一个选项对象。
    Vue.component('todo-item', {
        //该组件作为html元素可以接收的属性。
        props: ['todo'],
        //组件使用的html模板
        template: '<li>{{ todo.text }}</li>'
    })
    ```
    现在可以将组件名作为html元素使用：
    ```html
    <ol>
      <!--组件实例话-->
      <todo-item></todo-item>
    </ol>
    ```
* 组件之间数据作用域是隔开的，父子组件通过`props`通信

## 3.2 Data and Methods
* vue实例被创建时，会将`data`对象的属性添加到vue实例上和响应式系统中，即这些属性改变，视图也会改变。而之后vue实例添加的属性不会存在于响应式系统中。
* 除了选项对象提供的`data`,`methods`，vue实例也有一些有用的属性和方法，以`$`为开头。部分如下：
    * `$data`：选项对象中的data
    * `$el`：挂载在DOM中的html元素

## 3.3 生命周期
![](.Vue/aHR0cHM6Ly92dWVqcy5vcmcvaW1hZ2VzL2xpZmVjeWNsZS5wbmc.png )
vue允许在vue实例生命周期的某个阶段（如红色方框所示）上挂载用户自定义的函数，称为hook，如：

```javascript
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` points to the vm instance
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```

# 四 模板语法
* 即html模板，允许你声明式的绑定渲染后的DOM到vue实例的数据上。
* vue会编译模板到虚拟DOM渲染函数中。结合响应式系统，当vue实例状态改变时，vue会智能地计算最少的组件去渲染和应用最少的DOM操作。

## 4.1 插值
即将vue实例的数据插入到模板中，也称为数据绑定。
### 4.1.1 文本插值
数据被双花括号围绕`{{data}}`。如
```html
<span>Message: {{ msg }}</span>
```
这里的数据msg，来自选项对象中的`data`和？？谁？每当msg改变，view会被更新。

当使用`v-once`指令时，该元素只会插值一次：
```html
<span>Message: {{ msg }}</span>
```

### 4.1.2 Raw HTML
插入生的html，不被vue解析。如
```html
<span v-html="rawHtml"></span>
```
此时`span`的`innerHTML`使用属性rawHtml替换。

### 4.1.3 属性
使用指令`v-bind`,在需要插入数据的html属性上加上该指令。

### 4.1.4 表达式
前面都是绑定一些简单的属性值，也可以绑定表达式。表达式可以使用一些vue规定的全局变量如`Math`、`Date`，不能是自定义的全局变量。


## 4.2 指令
指定是特殊的属性，以`v-`开头。指令属性值期待一个js表达式（除了for）。指令的作用是当表达式值改变时，运用这些副作用（DOM的改变）到DOM中。

### 4.2.1 参数
指令是有参数的，指令与参数冒号分隔，如
```html
<a v-bind:href="url"> ... </a>
```
`v-bind`指令的参数是`href`，即将`url`绑定到`href`属性上。

### 4.2.2 动态参数
指定的参数可以是动态给出的，以`[]`为标识，js表达式给出参数值，如：
```html
<a v-bind:[attributeName]="url"> ... </a>
```
这里的`attributeName`是js表达式，表达式值必须为string，null表示删除绑定。

限制：`[]`内不能存在`spaces, quotes, <, >, / or =`

### 4.2.3 Modifiers
modifiers是指令的特殊后缀，以`.`表示，指示指令绑定的一些行为。如：
```html
<form v-on:submit.prevent="onSubmit"> ... </form>
```
表示触发事件时也执行`event.preventDefault()`

貌似可以叠加多个Modifiers

## 4.3 缩写
有的指令有缩写，如
* `v-bind`的缩写：`:`
* `v-on`的缩写：`@`
* `v-slot`的缩写：`#`

此时不需要冒号分隔指令与参数。

# 五 computed、watch
* **computed**：`computed`中的属性是`data`中属性进过计算后得到的属性。
    * `computed`中属性是一个`get`方法，也可以额外设置`set`方法。
    * `computed`中属性会放入vue实例中。
    * 一个例子：
    ```javascript
    var vm = new Vue({
      el: '#demo',
      data: {
        firstName: 'Foo',
        lastName: 'Bar'
      },
      computed: {
        fullName: function () {
          return this.firstName + ' ' + this.lastName
        }
      }
    })
    ```
    * 含有set方法的computed中属性例子：
    ```javascript
    // ...
    computed: {
      fullName: {
        // getter
        get: function () {
          return this.firstName + ' ' + this.lastName
        },
        // setter
        set: function (newValue) {
          var names = newValue.split(' ')
          this.firstName = names[0]
          this.lastName = names[names.length - 1]
        }
      }
    }
// ...
    ```
* **方法**：模板中也可以执行方法，达到和computed属性同样的效果。但computed属性会根据它用到的依赖缓存起来，只有依赖改变时才重新计算。而方法在每一次re-rende发生时会执行。
* `watch`：监听vue实例属性，改变时调用回调函数。`data`中一些属性的改变是基于其他属性的，可在`watch`中配置，如：
    ```javascript
    var vm = new Vue({
      el: '#demo',
      data: {
        firstName: 'Foo',
        lastName: 'Bar',
        fullName: 'Foo Bar'
      },
      watch: {
        firstName: function (val) {
          this.fullName = val + ' ' + this.lastName
        },
        lastName: function (val) {
          this.fullName = this.firstName + ' ' + val
        }
      }
    })
    ```
    上面配置了firstName和lastName改变时如何影响fullName改变。

# 六 class和style数据绑定
html的class和style都是属性，因此可以使用`v-bind`来绑定vue属性，并且不仅string、还可以绑定object or array到html属性上。
## 6.1 class数据绑定
* 文本绑定：
    ```html
    <div v-bind:class="activeClass"></div>
    ```
    这里的activeClass为vue实例属性。
* 对象绑定语法：  
    * 例子一
    ```html
    <div v-bind:class="{ active: isActive }"></div>
    ```
    对象的key为class名，对象的vlue为boolean值，true则class存在。这里的value值是vue对应的属性值。注意指令的值是表达式，可以计算的。
    * 例子二
    可以与HTML的class混合使用
    ```html
    <div
      class="static"
      v-bind:class="{ active: isActive, 'text-danger': hasError }"
    ></div>
    ```
    * 例子三
    可直接指定vue实例属性名：
    ```html
    <div v-bind:class="classObject"></div>
    ```
* 数组绑定语法：    
    ```html
    <div v-bind:class="[activeClass, errorClass]"></div>
    ```
    数组中元素都是vue实例属性，class属性值；想要类不存在，修改属性为`""`即可：
    ```html
    <div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
    <!--或者混合上面的对象绑定语法-->
    <div v-bind:class="[{ active: isActive }, errorClass]"></div>
    ```
* 涉及组件  
    组件内模板中绑定class，是绑定组件实例的属性。但下面的例子中：
    ```html
    <my-component v-bind:class="{ active: isActive }"></my-component>
    ```
    这里绑定的是父组件的实例属性。并且该class会被添加到组件模板的根元素上。

## 6.2 styles数据绑定
* 对象绑定语法  
    ```html
    <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
    ```
    对象的key是样式名，使用驼峰命令法；value是vue实例属性。
* 数组语法      
    数组允许绑定多个vue实例的style对象
    ```html
    <div v-bind:style="[baseStyles, overridingStyles]"></div>
    ```
* 自动添加前缀      
    有些样式在不同浏览器中有不同的前缀的，vue会自动添加。
* 一些样式的属性也有前缀，但需要自己手动添加，略。

# 七 条件性渲染
## 7.1 v-if
* 含有`v-if`指令的元素，只有在属性值为true时才会被渲染。
* 如果需要一个`v-if`指令条件渲染多个html元素，可使用`template`包裹起来：
    ```html
    <template v-if="ok">
      <h1>Title</h1>
      <p>Paragraph 1</p>
      <p>Paragraph 2</p>
    </template>
    ```
* 可像其他语言一样，有`v-else`,`v-else-if`
* vue实例数据改变时，会导致虚拟DOM重新渲染。但为了提高效率，虚拟DOM应用到DOM时，会复用已有的元素。使用`key`属性阻止这里行为：
    ```html
    <template v-if="loginType === 'username'">
      <label>Username</label>
      <input placeholder="Enter your username" key="username-input">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Enter your email address" key="email-input">
    </template>
    ```
    但label还是会被复用
## 7.2 v-show
含`v-show`的元素还是会被渲染到dom中，但会条件性的选择是否显示它，使用了css的`display`属性。因此隐藏或显示后，事件监听器还是存在的。

# 八 list渲染
## 8.1 v-for与数组
使用方式如下
* `v-for`可以将数组映射为Elements，最基本形式如下：
	```html
	  <li v-for="item in items">
	    {{ item.message }}
	  </li>
	```
	items为vue实例的数组属性，也可以是一个表达式；item为数组中的一项。然后可以使用`{{data}}`进行文本插值。
* 含有索引
	```html
	  <li v-for="(item, index) in items">
		{{ index }} - {{ item.message }}
	  </li>
	```
* 使用`of`，而不是`in`
	```html
	<div v-for="item of items"></div>
	```
## 8.2 v-for与对象
* 基本形式：
	```html
	  <li v-for="value in object">
	    {{ value }}
	  </li>
	```
	会遍历object的所有可遍历属性，value表示对象的属性。
* 取出属性名：
	```html
	<div v-for="(value, key) in object">
	  {{ key }}: {{ value }}
	</div>
	```
* 再添加索引：
	```html
	<div v-for="(value, key, index) in object">
	  {{ index }}. {{ key }}: {{ value }}
	</div>
	```
* 范围遍历
	```html
	<div>
	  <span v-for="n in 10">{{ n }} </span>
	</div>
	```
## 8.3 key
这里再次强调一下，虚拟DOM应用到DOM时会patch/reuse元素，达到对DOM的最少操作。元素（如input）或组件的内部状态可能会被保存，使用`key`属性可以将防止这种优化，如：
```html
<div v-for="item in items" :key="item.id">
  <!-- content -->
</div>
```
## 8.4 数组中元素改变探测
>对下面的话，做个总结：简而言之，vue检测不到`data`中数组属性中元素的修改，需要其他手段。对于对象也是同样的道理

我们知道，选项对象的`data`中的属性会被添加到响应式系统，这些属性的改变会造成视图的重新渲染。但如果`data`中存在**数组属性**，对**数组元素**的修改不会造成数据本身引用地址的改变（参考c++引用），因此vue不能通过数组属性检查数组元素是否改变。检测方法如下：
* 可变方法：数组操作通常是调用可变方法进行的，vue代理（wrap）了这些方法以检测数组元素的更改。这些方法有：push()、pop()、shift()、unshift()、splice()、sort()、reverse()
* 替换数组属性：有些方法不会改变原有数组，而是返回新数组，我们可以直接将新数组替换原有数组：
	```javascript
	example1.items = example1.items.filter(function (item) {
	  return item.message.match(/Foo/)
	})
	```
	替换数组属性不会造成视图全部更新，有优化的~
* 其他：由于javascript限制，直接使用索引修改数组值，和直接修改数组属性，vue是检测不到的。解决方法如下：
	* 使用`Vue.set()`来设置属性（索引）值
		```javascript
		Vue.set(vm.items, indexOfItem, newValue)
		```
		>vue实例也有该方法`vm.$set()`
	* 使用可变方法`splice()`
		```javascript
		vm.items.splice(indexOfItem, 1, newValue)
		```
		它在删除元素的同时也在对应位置添加了`newValue`

## 8.5 对象中属性更改检测
理由同上，解决方法
* 使用`set`方法
* 覆盖原对象

## 8.6 其他
* `v-for`也可以使用`<template>`元素聚合html元素块。
* 当`v-for`与`v-if`一起作用到同一个元素中，`v-for`优先级高
* 当`v-for`与组件搭配使用时，`key`属性必须加上，保证组件的更改随视图更新而更新。组件之间的数据作用域是隔开的，如果父组件想将`v-for`的索引值传入子组件，需绑定子组件的属性，子组件的属性由选项对象的`props`设置。

# 九 事件处理
`v-on`绑定的事件在虚拟DOM删除（real-DOM中reuse），事件也会被清除。
## 9.1 事件处理器
`v-on`指令监听DOM事件，事件触发时执行js代码。
* 指令值为js代码：
	```html
	<button v-on:click="counter += 1">Add 1</button>
	```
	指令值的可用`$event`表示DOM原生事件，如：
	```html
	<!--这里调用了方法，传入$event-->
	<button v-on:click="methodName("arg", $event)">
	  Submit
	</button>
	```
* 指令值为方法名，方法原型可以直接接受event

## 9.2 事件modifiers
之前说了，一些指令拥有modifier，`v-on`也有。

>参考：[Event modifiers](https://vuejs.org/v2/guide/events.html#Event-Modifiers)

## 9.3 键值modifiers
当只监听某个键值时，可以使用key modifiers。如：
```html
<!-- only call `vm.submit()` when the `key` is `Enter` -->
<input v-on:keyup.enter="submit">
```

* 键值来自[KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)。并且使用**kebab-case**格式，如`PageDown`写成`page-down`。
* 由于兼容性问题，vue提供（覆盖）了一些键名，如`.enter`
* modifiers还可用`keyCode`，如监听回车事件：
	```html
	<input v-on:keyup.13="submit">
	```
* 系统键值modifiers、鼠标按键modifers：[System Modifier Keys](https://vuejs.org/v2/guide/events.html#System-Modifier-Keys)

# 十 Form Input绑定
`v-model`用于对表单元素进行双向绑定，略！但记住：
```html
<input v-model="searchText">
```
等于
```html
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```
如果是组件，则等于：
```html
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```
这里的`$event`是组件发出自定义事件时传的参数。与`v-on`的`$event`不同。
# 十一 组件基础
* 尽管组件本质上是一个Vue实例。但在html中仍不能单独使用，需要放入vue实例的html元素中；组件通过js代码实例化可以作为根组件。
* Vue实例有特定于自己的根选项：`el`
* 组件的`data`选项必须是一个函数，以致于每个组件实例都有自己的数据拷贝。
* 组件注册：全局和局部。全局组件通过`Vue.component()`注册。全局组件可以作为根组件
* `props`是组件的自定义属性，组件实例后会作为实例的属性而存在

## 11.1 自定义事件与监听
子组件中使用`vm.$emit( eventName, […args] )`自定义事件并发出，arg是伴随事件的额外数组参数。
```html
<!--位于组件blog-post中，点击发送事件，带着参数-->
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
父组件可以监听该事件
```html
  <!--父组件中-->
<blog-post
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```
这里的`$event`代表`$emit()`方法的参数数组中的第一个参数。

或者使用事件监听器：
```html
<blog-post
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```
```javascript
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```
事件处理器的参数可以有多个，与`$emit()`方法的参数一一对应。

## 11.2 v-model与组件
上面谈到过
```html
<customer-input v-model="searchText"></customer-input>
```
等于
```html
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```
因此若子组件想要实现`v-model`的数据双向绑定，可进行如下设计：
```javascript
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```
------------
如果想实现组件其他属性的双向绑定，需要`model`选项更改`v-model`绑定的属性和事件，如：
```javascript
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```
现在可以使用`v-model`绑定到组件的checked属性上
```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```

## 11.3 slots
html中，组件中含有内容时，组件可通过`<slot>`获取。如：
```html
<alert-box>
  Something bad happened.
</alert-box>
```
```javascript
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```

## 11.4 动态组件
动态切换组件，使用`is`属性，如：
```html
<div id="app">
   <component  v-bind:is="which"></component >
   <select name="" id="" v-model="which">
       <option>my-content1</option>
       <option>my-content2</option>
       <option>my-content3</option>
   </select>
</div>
```
```javascript
    let vm=new Vue({
        el:"#app",
        data:{
            which:"my-content1"
        },
        components:{
            "my-content1":{
                template:`<div>content1</div>`
            },
            "my-content2":{
                template:`<div>content2</div>`
            },
            "my-content3":{
                template:`<div>content3</div>`
            }
        }
    })
```
每次切换时，vue会重新生成组件实例，那么之前的状态不会被保存，需要使用`<keep-alive>`内置组件包裹它，使模板缓存起来：
```html
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

## 11.5 DOM模板解析问题
在html元素中，`ul`,`ol`,`table`,`select`规定了什么元素能够存在，如`li`,`tr`,`option`。解决办法是使用`is`属性。

但在其他模板中无这种问题：
* 字符串模板，如`template:'....'`
* 单组件文件`.vue`
* `<script type="text/x-template">`


# 十二 组件注册
## 12.1 全局注册
```javascript
Vue.component('my-component-name', { /* ... */ })
```
第一个参数为组件名，第二个参数为选项对象。全局注册的组件，可以直接在其他组件中使用。

## 12.2 局部注册
创建Vue实例时，可将组件传给选项对象的components数组，如：
```javascript
//选项对象
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }

new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```
并且该组件只能在注册的父组件中使用。
## 12.3 组件名
* kebab-case（推荐）：注册组件名如`my-component-name`，可以在html和模板中直接使用。
* PascalCase：注册组件名如`MyComponentName`，此时在html中只能使用对应的kebab-case命令，但在模板中两则都可使用。(即存在自动转换过程)

>注意，html中元素和属性名都大小写不敏感。

# 十三 Props
props选项属性定义组件作为html元素收到的属性。属性的命令规则同上。

属性可以为数组或对象。对象可以额外设置属性类型、默认值、是否必须、验证函数。

属性值常与`v-bind`一起使用。

如果在组件上使用props未定义的属性，那么属性会被添加到组件的根元素上。如果根元素已定义该属性，则被覆盖；`style`,`class`属性除外，它会被合并在一起。

`inheritAttrs: false`会阻止未定义属性作用在组件根元素上的行为，而`$attrs`接收未定义属性的赋值，因此此时可以将它手动绑定到组件中的一个元素上，如：
```javascript
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```
>注意，都不会影响`class`和`style`属性。

参考：[props](https://vuejs.org/v2/api/#props);

# 十四 事件
略

# 十五 slots
* 组件在作为html使用时，如果组件元素含有内容，默认行为是忽略它，在组件的模板中使用`<slot>`元素可以获取内容。

* 组件元素中的文本插值的变量作用域位于父组件

* 模板中`<slot>`元素中可有默认内容，组件元素中可覆盖这些内容。

* 当有多个`<slot>`元素时，使用元素的`name`属性来区分，允许其中一个`<slot>`无`name`值，则默认`default`；那么组件元素的内容中需要使用`<template>`围绕部分内容，`v-slot`指令指定对应`slot`元素。如：
	**模板**
	```html
	<div class="container">
	  <header>
	    <slot name="header"></slot>
	  </header>
	  <main>
	  	<!--默认名default-->
	    <slot></slot>
	  </main>
	  <footer>
	    <slot name="footer"></slot>
	  </footer>
	</div>
	```
	**父组件模板**
	```html
	<base-layout>
	  <template v-slot:header>
	    <h1>Here might be a page title</h1>
	  </template>
	
	  <template v-slot:default>
	    <p>A paragraph for the main content.</p>
	    <p>And another one.</p>
	  </template>
	
	  <template v-slot:footer>
	    <p>Here's some contact info</p>
	  </template>
	</base-layout>
	```
	同样允许无`<template>`元素围绕的部分存在，对应default slot
	```html
	<base-layout>
	  <template v-slot:header>
	    <h1>Here might be a page title</h1>
	  </template>
	
	  <p>A paragraph for the main content.</p>
	  <p>And another one.</p>
	
	  <template v-slot:footer>
	    <p>Here's some contact info</p>
	  </template>
	</base-layout>
	```
	
* ~~`<slot>`元素中的属性会被绑定到一个对象中，组件元素中`v-slot`可以获取该对象，实现了父组件中的组件元素使用组件内的数据。如：~~
	
	所有绑定到`<slot>`元素的属性上的对象, 都会被存入一个对象中(如下面的`slotProps`), 在父组件的模板中可以通过`v-slot`指令取出, 实现了子组件通过`slot`元素为父组件提供数据. 例子如下:
	
	**模板**
	
	```html
	<!-- 有一个user属性 -->
	<span>
	  <slot v-bind:user="user">
	    {{ user.lastName }}
	  </slot>
	</span>
	```
	**父组件模板中**
	```html
	<current-user>
	  <template v-slot:default="slotProps">
	    {{ slotProps.user.firstName }}
	  </template>
	</current-user>
	```
	当只有一个slot元素时，可缩写成：
	```html
	<current-user v-slot:default="slotProps">
	  {{ slotProps.user.firstName }}
	</current-user>
	```
	也可省略default
	```html
	<current-user v-slot="slotProps">
	  {{ slotProps.user.firstName }}
	</current-user>
	```
	上面传入的slotProps对象含有slot元素的属性值，可以使用es5的解构语法获得单个属性：
	```html
	<current-user v-slot="{ user }">
	  {{ user.firstName }}
	</current-user>
	```
	
* `v-slot`的参数可以是动态的，如`v-slot:[dynamicSlotName]`

* `v-slot:`的缩写为`#`，使用时后面必须存在参数。

# 十六 例子
## 16.1 指令与vue实例
```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
```
```javascript
//创建vue实例时，需要传入选项对象
var app5 = new Vue({
  //绑定的element
  el: '#app-5',
  //data里的属性最终成为vue实例的属性
  data: {
    message: 'Hello Vue.js!'
  },
  //methods中的属性最终成为vue实例的方法，因此this指向vue实例
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```
# 十七 其他
## 17.1 获取元素
* `$root`：访问组件的根实例
* `$parent`：访问子组件的父组件
* `$refs`：子元素或子组件上添加`ref`属性，指定引用id，可通过`$refs`来获取该DOM元素或组件实例。
* 	`$el`：组件的根DOM元素
## 17.2 .sync
用于双向绑定属性的，与`v-model`类似。

如父组件绑定到子组件的`title`属性上，当子组件发出`update:title`事件时：
```javascript
this.$emit('update:title', newTitle)
```
父组件更新`title`值：
```html
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```
可简写为：
```html
<text-document v-bind:title.sync="doc.title"></text-document>
```
>可以发现，`.sync`与`v-model`监督的事件不同，`.sync`监听的事件前有`update:`前缀，`v-model`默认监听`input`事件。
# 参考
* [vue api](https://vuejs.org/v2/api/#Options-Data)