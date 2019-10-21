# Validation

* 未与Web层绑死, 可用在其他地方
*  JSR-303 Bean Validation API  被全面支持

> 是对Bean的校验, 参数本身校验貌似未支持, 这个可以靠基本类型验证, 暂时以后再了解;

> 参考
>
> * [Spring Validation](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#validation-beanvalidation)
> * [SpringMVC validation完成后端数据校验（较全面）](https://blog.csdn.net/m0_37589327/article/details/78648328)  : 写的还行吧

# 资源获取

`ResourceLoader`提供的`getResource`方法, 可以获取代表资源的`Resource`. 但资源是否真的存在, 需要`Resource.exists()`检测.

`ResourceLoader`会自动检查url是指向文件`file://C:/test.dat`还是classpath下`classpath:test.dat`

> 常用`ResourceLoader`的实现类`DefaultResourceLoader`

