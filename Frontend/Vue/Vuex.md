# 介绍

## 引言

在Vue组件的开发中,组件被抽象成了三部分:

![img](.Vuex/flow.png)

**状态**被渲染成视图, **视图**接受用户操作触发动作, 动作又回来改变状态. 数据在其中单向流动.

一个页面由多个APP组成, 可能会出现这样的情景:

1. 多个视图的渲染依赖于相同的状态

2. 不同视图触发的动作会改变相同的状态.

也就是说, 此时我们需要一个全局状态, 而组件间独立性较高, 没有Vuex之前有如下方案:

1. 状态位于父组件中, 靠属性`props`将状态传入子组件中, 解决问题一; 子组件通过事件通知父组件改变状态, 解决问题二.
2. 通过session存取变量, 自己维护状态

方案一不适合组件间不独立性不高,有一定耦合度的场景; 方案二需要自己实现状态管理器, 要写好相当于造轮子. 而Vuex已经提供了它自己的解决方案, 即全局的状态管理器.

> Vuex仅是一个状态管理器, 要维护一个session数据, 还是要存入浏览器的session中

通过Vuex, 组件树就耦合成了一个大的组件, 因此使用前需要权衡组件的耦合度和独立度.

## 核心概念

* store代表状态管理器, 含有状态, 和操作状态的方法
* state表示全局状态
* getter通过全局状态计算得到的全局状态.
* mutation是改变状态的方法, 仅支持同步操作
* action同样是改变状态的方法, 但支持异步操作
* module模块也含有状态和操作方法, 主要用于分割名字空间.

组件中不能直接对状态state操作, 因为这样会造成视图无响应. 同步操作需要提交mutation(变化), 异步操作需要提交action(动作).

![vuex](.Vuex/vuex.png)

## 安装

* 安装

  * 下载包

    ```bash
    npm install vuex --save
    ```

  * 注册为Vue插件

    ```javascript
    import Vue from 'vue'
    import Vuex from 'vuex'
    
    Vue.use(Vuex)
    ```

# 使用

## Vuex.Store

* 创建一个容器

  ```javascript
  const store = new Vuex.Store({
    state: {
      count: 0
    },
    mutations: {
      increment (state) {
        state.count++
      }
    }
  })
  ```

  > 作为参数的对象可以定义状态,方法,模块等.

* 组件中使用

  * 普通方式: 直接使用`store`对象访问状态和方法, 

    * 获取对应属性: `store.state.XXX`,`store.getters.XXX`
  
    * 修改状态:
  
      * 同步修改

        ```javascript
      //参数为action方法名
        store.commit("increment");
        ```

      * 异步修改
  
        ```javascript
        //参数同上
        store.dispatch("increment");
        ```
  
    * 例子
  
        ```javascript
        store.commit('increment')

        console.log(store.state.count) // -> 1
        ```
  
  * 根组件启动`store`选项
  
    * 可以免去所有子组件都需要引入`store`对象的过程, 子组件可通过`this.$store`获取该对象.
    * 使用`mapXXX`函数简化状态方法的映射过程.
  

## state

应用的全局状态会存入`store`的`state`对象中.

有如下几种使用方式:

* 在`computed`属性中暴露全局状态

  ```javascript
  computed: {
      count () {
          return store.state.count
      }
  }
  ```

  如果Vue实例启用了`store`选项, 则可通过`this.$store`获取`store`对象.

* `mapState()`简化了获取全局状态的过程.

  ```javascript
  computed: mapState({
      //比上面更简洁
      count: state => state.count,
  
      //count直接指定状态
      countAlias: 'count',
  
      //访问组件的局部状态时必须用普通函数
      countPlusLocalState (state) {
          return state.count + this.localCount
      }
  })
  ```

  > 函数的第一个参数为`state`对象

  若不更改状态名, 可直接使用字符串数组

  ```javascript
  computed: mapState([
    // map this.count to store.state.count
    'count'
  ])
  ```

  若还想定义局部`computed`属性, 可使用Spread语法

  ```javascript
  computed: {
    localComputed () { /* ... */ },
    // mix this into the outer object with the object spread operator
    ...mapState({
      // ...
    })
  }
  ```

## getters

通过全局状态计算得到的**全局状态**需要放入`store`的`getters`对象中

```java
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

* `getters`状态需要通过`store.getters`访问

* 参数, 第二个参数可以是`getters`
* vue组件中使用与`state`一致

* `getter`可以返回一个有参数的函数

  ```javascript
  getters: {
    // ...
    getTodoById: (state) => (id) => {
      return state.todos.find(todo => todo.id === id)
    }
  }
  ```

  ```JavaScript
  store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
  ```

* `mapGetters`映射`getters`到局部属性中.

  ```JavaScript
  he mapGetters helper simply maps store getters to local computed properties:
  
  import { mapGetters } from 'vuex'
  
  export default {
    // ...
    computed: {
      // mix the getters into computed with object spread operator
      ...mapGetters([
        'doneTodosCount',
        'anotherGetter',
        // ...
      ])
    }
  }
  ```

  如果使用不同的名字

  ```JavaScript
  ...mapGetters({
    // map `this.doneCount` to `this.$store.getters.doneTodosCount`
    doneCount: 'doneTodosCount'
  })
  ```

  > 不知道传入额外参数的状态是如何映射的.

## mutations

直接修改Vuex的状态不会有reactive, 需要`store`中声明`mutations`对象后, 通过`store.commit`方法提交这个改变.

```javascript
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // mutate state
      state.count++
    }
  }
})
```

```javascript
store.commit('increment')
```

`store.commit`方法中可以传入第二个参数, `mutations`中同样在第二个参数中获得.

```javascript
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

```javascript
store.commit('increment', {
  amount: 10
})
```

对象式的提交

```javascript
store.commit({
  type: 'increment',
  amount: 10
})
```

```javascript
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

如果状态是对象, 当新增属性时, 需要特定的方法或者直接覆盖状态, 才会触发响应reactive

```javascript
state.obj = { ...state.obj, newProp: 123 }
```

组件中使用

* `this.$store.commit('xxx')`

* `mapMutations`

  ```javascript
  import { mapMutations } from 'vuex'
  
  export default {
    // ...
    methods: {
      ...mapMutations([
        'increment', // map `this.increment()` to `this.$store.commit('increment')`
  
        // `mapMutations` also supports payloads:
        'incrementBy' // map `this.incrementBy(amount)` to `this.$store.commit('incrementBy', amount)`
      ]),
      ...mapMutations({
        add: 'increment' // map `this.add()` to `this.$store.commit('increment')`
      })
    }
  }
  ```

函数内必须是同步的!

## actions

actions函数允许异步, 第一个参数为context, 该对象同样提供了`state`,`getters`和`commit`对象, 同时也提供了`dispatch`方法用于执行action函数. 但`context`并不是`store`对象.

```javascript
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

也可有参数

```javascript
// dispatch with a payload
store.dispatch('incrementAsync', {
  amount: 10
})

// dispatch with an object
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

组件中使用, 同上, 除了映射函数为`mapActions`

```javascript
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // map `this.increment()` to `this.$store.dispatch('increment')`

      // `mapActions` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // map `this.add()` to `this.$store.dispatch('increment')`
    })
  }
}
```

`store.dispatch()`返回的是action方法返回的`promise`对象.

## modules

每个模块都可以有`state`,`mutations`,`actions`,`getters`

```javascript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> `moduleA`'s state
store.state.b // -> `moduleB`'s state
```

... 略...

# todo

* [vuex-module-decorators](https://championswimmer.in/vuex-module-decorators/) 通过装饰器的方式使用Vuex
* Vuex Module完善





































