# 介绍

Vue3 ( 即[vue-next](https://github.com/vuejs/vue-next) ) 当前还处于预览版, 但很多提案[rfcs](https://github.com/vuejs/rfcs)已经出来了. 

其中最重要的提案就是[Function-based Component API #42](https://github.com/vuejs/rfcs/pull/42)

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

  



# 学个锤子!

目前为止, Vue3还未出来, Composition API提供了Vue2.x的插件, 在NuxtJs中有简单Demo. 

实践过程中遇到了Nuxt框架自身的类型检查错误, 解决之后, 也不知道怎么在Composition API中使用子组件....

呸呸呸! 学React去!!!





> 参考 [Composition API RFC](https://vue-composition-api-rfc.netlify.com/#summary) , 包含目前API最新的提案内容和API使用方法

# 待学

[Vue3 Composition-API](https://mp.weixin.qq.com/s/mCZK_KYZFmhZtlscHyMLiw)