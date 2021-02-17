# Start

## 安装

```shell
npm install -g @vue/cli
```

## Intro

1.  应用实例 & 根组件

   ```js
   const RootComponent = { /* options */ } // 根组件实例
   Vue.createApp(RootComponent) // 创建应用实例
     .component('SearchInput', SearchInputComponent) // 注册应用实例下的全局组件
     .directive('focus', FocusDirective) // 注册指令
     .use(LocalePlugin) // 注册插件
     .mount('#app') // 挂载, 返回根组件实例
   ```

* 组件实例的属性

  * `data`, `methods`, `props`, `computed`, `inject`, `setup`的属性都会挂载到组件实例上

  * 组件模板可以访问到组件实例的属性
  * Vue还提供了内置属性, 以`$`为前缀, 如`$attrs`, `$emit`.
  * 以`_`为前缀的名字为保留名字, 用户不允许使用.

* 生命周期

  ![lifecycle](.Vue3/lifecycle-1608959619713.png)



## Template Syntax

* 用于绑定组件实例的属性到DOM上
* 





# Reactivity

## 原理

```javascript
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      return reactive(value)
    } else {
      return value
    }
  }
  set(target, key, value, receiver) {
    trigger(target, key)
    return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

1. 响应式对象由代理实现
2. `get()`方法记录依赖, `set()`方法出发更新
3. 组件首次渲染时, 会访问涉及的所有响应式对象, 出发`get()`的记录依赖树.
4. `computed`对象无需指定依赖, 无副作用的`watch`需要指定监听的响应式对象, 原因见3.
5. 代理只对对象有用, 所以vue3提出了`ref`的概念, 用于基本类型变量.

# 过渡 & 动画

* 原理

  `transition`或`transition-group`组件中指定`name`后, 元素插入或删除时, 会被作用不同的class. 

  我们只需要定义这些class的样式即可

* 组件

  * `transition`
    * 作用于单元素/组件, 或者多个元素/组件, 但同一时间内仅出现一个元素/组件(互斥)
    * 不产生新的wrapper元素

  * `transition-group`
    * 作用于多个元素/组件, 每个元素有唯一的`key`值
    * 可以产生新的wrapper元素.
    * 支持元素位置改变时的过度效果.

* 使用
  * [transition](https://v3.cn.vuejs.org/api/built-in-components.html#transition)
  * [transition-group](https://v3.cn.vuejs.org/api/built-in-components.html#transition-group)

# Vue Router

* 获取`route`, `router`

  ```javascript
  const router = useRouter()
  const route = useRoute()
  ```

* `route`是响应式对象, 可监听, 以获取变更后的动态参数值

  ```javascript
  import { useRoute } from 'vue-router'
  
  export default {
    setup() {
      const route = useRoute()
      const userData = ref()
  
      // fetch the user information when params change
      watch(
        () => route.params,
        async newParams => {
          userData.value = await fetchUser(newParams.id)
        }
      )
    },
  }
  ```

* 组件路由拦截器

  ```javascript
  import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'
  
  export default {
    setup() {
      // same as beforeRouteLeave option with no access to `this`
      onBeforeRouteLeave((to, from) => {
        const answer = window.confirm(
          'Do you really want to leave? you have unsaved changes!'
        )
        // cancel the navigation and stay on the same page
        if (!answer) return false
      })
  
      const userData = ref()
  
      // same as beforeRouteUpdate option with no access to `this`
      onBeforeRouteUpdate(async (to, from) => {
        // only fetch the user if the id changed as maybe only the query or the hash changed
        if (to.params.id !== from.params.id) {
          userData.value = await fetchUser(to.params.id)
        }
      })
    },
  }
  ```

  

# 其他

## idea文件模板

```vue
<template>
  #[[$END$]]#
</template>

<script lang="ts">
import {defineComponent} from "vue";
export default defineComponent ({
  name: "${NAME}"
})
</script>

<style lang="scss" scoped>

</style>
```

