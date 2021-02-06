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


