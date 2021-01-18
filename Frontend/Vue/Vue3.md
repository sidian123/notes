# Start

## 安装

使用或更新Vue CLI到v4.5及以上版本.

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

















# Composition API

## setup

* 介绍

  `setup()`是组件创建的新方式, 和Composition API的入口. 

  > 可以理解成MVC的视图层.

* 调用顺序

  当创建组件时, 先属性Props初始化 -> `setup()`调用 -> `beforeCreate()`

  > 实际上, Composition API基于原先的Options API. 因此, 这里的属性初始化指, 以Composition API方式声明组件时的属性初始化.

* 模板中使用

  `setup()`返回对象的属性将融合到模板的渲染环境中

  ```javascript
  <template>
    <div>{{ count }} {{ object.foo }}</div>
  </template>
  
  <script>
  import { ref, reactive } from 'vue'
  
  export default {
    setup() {
      const count = ref(0)
      const object = reactive({ foo: 'bar' })
  
      // expose to template
      return {
        count,
        object
      }
    }
  }
  </script>
  ```

  > ref在模板中将自动unwrap

* 直接返回渲染函数或JSX

  略

* 参数

  







> 参考 [Composition API RFC](https://vue-composition-api-rfc.netlify.com/#summary) , 包含目前API最新的提案内容和API使用方法

# 参考

* [Vue3 Composition-API](https://mp.weixin.qq.com/s/mCZK_KYZFmhZtlscHyMLiw)
* https://zhuanlan.zhihu.com/p/337871096