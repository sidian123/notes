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

# util

Spring的`util`包下, 含有各种工具类, 如

*  [Base64Utils](http://localhost:63342/Blog/spring-core-5.1.9.RELEASE-javadoc.jar/org/springframework/util/Base64Utils.html) : A simple utility class for Base64 encoding and decoding. 
*  [StringUtils](http://localhost:63342/Blog/spring-core-5.1.9.RELEASE-javadoc.jar/org/springframework/util/StringUtils.html) : Miscellaneous [`String`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html?is-external=true) utility methods. 
*  [DigestUtils](http://localhost:63342/Blog/spring-core-5.1.9.RELEASE-javadoc.jar/org/springframework/util/DigestUtils.html) : Miscellaneous methods for calculating digests.
* ...

> 参考[Package org.springframework.util](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/package-summary.html)

# beans

含与Bean相关的接口与类

*  [BeanUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/BeanUtils.html) : Static convenience methods for JavaBeans: for instantiating beans, checking bean property types, copying bean properties, etc. 
  * `copyProperties()`: bean间拷贝属性

> 参考[Package org.springframework.beans](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/package-summary.html)











