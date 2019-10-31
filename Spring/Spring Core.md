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

# 工具

## util

Spring的`util`包下, 含有各种工具类, 如

*  [Base64Utils](http://localhost:63342/Blog/spring-core-5.1.9.RELEASE-javadoc.jar/org/springframework/util/Base64Utils.html) : A simple utility class for Base64 encoding and decoding. 
*  [StringUtils](http://localhost:63342/Blog/spring-core-5.1.9.RELEASE-javadoc.jar/org/springframework/util/StringUtils.html) : Miscellaneous [`String`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html?is-external=true) utility methods. 
*  [DigestUtils](http://localhost:63342/Blog/spring-core-5.1.9.RELEASE-javadoc.jar/org/springframework/util/DigestUtils.html) : Miscellaneous methods for calculating digests.
* ...

> 参考[Package org.springframework.util](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/package-summary.html)

## beans

含与Bean相关的接口与类

*  [BeanUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/BeanUtils.html) : Static convenience methods for JavaBeans: for instantiating beans, checking bean property types, copying bean properties, etc. 
  * `copyProperties()`: bean间拷贝属性

> 参考[Package org.springframework.beans](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/package-summary.html)



# Cache

## 配置

* 启用缓存

  配置类上添加` @EnableCaching `注解

* Spring提供了Cache的抽象层, 而实现一般由第三方提供.

   如果未提供, SpringBoot则使用` ConcurrentHashMap `作为缓存. (不建议用在生产环境中)

* 关于并发

  抽象层未提供任何多线程的处理, 该功能取决于抽象层的实现.

* 配置缓存

  通过` CacheManager `可详细配置缓存

  > 该类的实现由第三方提供

  SpringBoot在自动装配`CacheManager`时, 提供了回调的入口, 来配置缓存:

  ```java
  @Bean
  public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
      return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {
          @Override
          public void customize(ConcurrentMapCacheManager cacheManager) {
              cacheManager.setAllowNullValues(false);
          }
      };
  }
  ```

  > 可以定义多个`Customizer`, Spring只会使用与当前缓存实现对应的`Customizer`

* 关于` CacheManager `和` Cache `

  `CacheManager`是`Cache`的管理器, `Cache`代表具体的一个缓存. 可以手动注入一个`Cache` Bean到容器中, `CacheManager`会自动管理它.

* 以Redis作为缓存
  * SpringBoot中, 若存在Spring Data Redis的Jar时, 会自动配置` RedisCacheManager `Bean
  * 手动注入` RedisCacheConfiguration `可以全面掌控Redis缓存的配置, 如配置解析器.

## 使用

> 因为没有深入的研究源码, 可能下面的讲解会穿插Redis的实现

### @Cacheable

* 介绍

  注解在方法或类上, 表示方法的结果是可缓存的.

* 切面逻辑

  访问方法时, 如果已存在待请求的资源, 则直接返回; 否则调用方法, 将结果存入到缓存, 再返回结果.

* Key生成

  默认将方法参数传给` KeyGenerator `, 由其生成Key. 可设置注解的`key`元素, 手动设置, 使用SpEL表达式. 具体使用见文档.

  > 默认生成方式不太符合一般使用, 请手动设置

* Redis实现细节

  * 真正存储在Redis中的Key为`<cacheNames>::<key>`, 如

    ```java
    @Cacheable(value="blog",key = "#a")
    public String cache1(String a){
        return "1";
    }
    ```

    则key可能是`blog::c`

  * Key用`StringRedisSerializer`序列化; Value用`JdkSerializationRedisSerializer`序列化











* `@CacheEvict`: Triggers cache eviction.
* `@CachePut`: Updates the cache without interfering with the method execution.
* `@Caching`: Regroups multiple cache operations to be applied on a method.
* `@CacheConfig`: Shares some common cache-related settings at class-level.





> 参考
>
> * [Cache Abstraction](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/integration.html#cache)
> * [Spring Boot Cache](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching)



# SpEL

## 引言

### 介绍

SpEL（Spring Expression Language），即Spring表达式语言，是比JSP的EL更强大的一种表达式语言。为什么要总结SpEL，因为它可以在运行时查询和操作数据，尤其是数组列表型数据，因此可以缩减代码量，优化代码结构。个人认为很有用。

### 用法

 SpEL有三种用法:

* 注解@Value中使用

  ```java
  //@Value能修饰成员变量和方法形参
  //#{}内就是表达式的内容
  @Value("#{表达式}")
  public String arg;
  ```

* XML配置

  ```xml
  <bean id="xxx" class="com.java.XXXXX.xx">
      <!-- 同@Value,#{}内是表达式的值，可放在property或constructor-arg内 -->
      <property name="arg" value="#{表达式}">
  </bean>
  ```

* 代码中使用`Expression`

  ```java
  import org.springframework.expression.Expression;
  import org.springframework.expression.ExpressionParser;
  import org.springframework.expression.spel.standard.SpelExpressionParser;
  import org.springframework.expression.spel.support.StandardEvaluationContext;
   
  public class SpELTest {
   
      public static void main(String[] args) {
   
          //创建ExpressionParser解析器
          ExpressionParser parser = new SpelExpressionParser();
          //获取表达式
          Expression exp = parser.parseExpression("表达式");
          //执行表达式，默认容器是spring本身的容器：ApplicationContext
          Object value = exp.getValue();
          
          /**如果使用其他的容器，则用下面的方法*/
          //创建一个虚拟的容器EvaluationContext
          StandardEvaluationContext ctx = new StandardEvaluationContext();
          //向容器内添加bean
          BeanA beanA = new BeanA();
          ctx.setVariable("bean_id", beanA);
          
          //setRootObject并非必须；一个EvaluationContext只能有一个RootObject，引用它的属性时，可以不加前缀
          ctx.setRootObject(XXX);
          
          //getValue有参数ctx，从新的容器中根据SpEL表达式获取所需的值
          Object value = exp.getValue(ctx);
      }
  }
  ```

## 表达式语法

### 字面值赋值

 ```xml
<!-- 整数 -->
<property name="count" value="#{5}" />
<!-- 小数 -->
<property name="frequency" value="#{13.2}" />
<!-- 科学计数法 -->
<property name="capacity" value="#{1e4}" />
<!-- 字符串  #{"字符串"} 或  #{'字符串'} -->
<property name="name" value="#{'我是字符串'}" />
<!-- Boolean -->
<property name="enabled" value="#{false}" />
 ```

>注：
>
>* 字面量赋值必须要和对应的属性类型兼容，否则会报异常。
>* 一般情况下我们不会使用 SpEL字面量赋值，因为我们可以直接赋值。

###  引用Bean、属性和方法

>（必须是public修饰的） 

```xml
<property name="car" value="#{car}" />
<!-- 引用其他对象的属性 -->
<property name="carName" value="#{car.name}" />
<!-- 引用其他对象的方法 -->
<property name="carPrint" value="#{car.print()}" />
```

### 运算符

* 算术运算符` +,-,*,/,%,^ `

  ```xml
  <!-- 3 -->
  <property name="num" value="#{2+1}" />
  <!-- 1 -->
  <property name="num" value="#{2-1}" />
  <!-- 4 -->
  <property name="num" value="#{2*2}" />
  <!-- 3 -->
  <property name="num" value="#{9/3}" />
  <!-- 1 -->
  <property name="num" value="#{10%3}" />
  <!-- 1000 -->
  <property name="num" value="#{10^3}" />
  
  ```

* 字符串连接符`+`

  ```xml
  <!-- 10年3个月 -->
  <property name="numStr" value="#{10+'年'+3+'个月'}" />
  ```

* 比较运算符` <(<),>(>),==,<=,>=,lt,gt,eq,le,ge `

  ```xml
  <!-- false -->
  <property name="numBool" value="#{10&lt;0}" />
  <!-- false -->
  <property name="numBool" value="#{10 lt 0}" />
  <!-- true -->
  <property name="numBool" value="#{10&gt;0}" />
  <!-- true -->
  <property name="numBool" value="#{10 gt 0}" />
  <!-- true -->
  <property name="numBool" value="#{10==10}" />
  <!-- true -->
  <property name="numBool" value="#{10 eq 10}" />
  <!-- false -->
  <property name="numBool" value="#{10&lt;=0}" />
  <!-- false -->
  <property name="numBool" value="#{10 le 0}" />
  <!-- true -->
  <property name="numBool" value="#{10&gt;=0}" />
  <!-- true -->
  <property name="numBool" value="#{10 ge 0}" />
  ```

* 逻辑运算符` and,or,not,&&(&&),||,! `

  ```xml
  <!-- false -->
  <property name="numBool" value="#{true and false}" />
  <!-- false -->
  <property name="numBool" value="#{true&amp;&amp;false}" />
  <!-- true -->
  <property name="numBool" value="#{true or false}" />
  <!-- true -->
  <property name="numBool" value="#{true||false}" />
  <!-- false -->
  <property name="numBool" value="#{not true}" />
  <!-- false -->
  <property name="numBool" value="#{!true}" />
  ```

* 条件运算符` ?true:false `

  ```xml
  <!-- 真 -->
  <property name="numStr" value="#{(10>3)?'真':'假'}" />
  ```

* 正则表达式` matches `

  ```xml
  <!-- true -->
  <property name="numBool" value="#{user.email matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+.[a-zA-Z]{2,4}'}" />
  ```

* 调用静态方法或静态属性

   通过 T() 调用一个类的静态方法，它将返回一个 Class Object，然后再调用相应的方法或属性 

  ```xml
  <!-- 3.141592653589793 -->
  <property name="PI" value="#{T(java.lang.Math).PI}" />
  ```

* 获取容器中的变量

  使用`#bean_id`获取. 其中, 有两个特殊变量

  * `this` 使用当前正在计算的上下文
  * `root` 引用容器的`root`对象

  ```java
   String result2 = parser.parseExpression("#root").getValue(ctx, String.class);  
   
          String s = new String("abcdef");
          ctx.setVariable("abc",s);
          //取id为abc的bean，然后调用其中的substring方法
          parser.parseExpression("#abc.substring(0,1)").getValue(ctx, String.class);
  
  ```

* 方法调用

  与Java代码没有什么区别，可见上面的例子
  可以自定义方法，如下： 

  ```java
  Method parseInt = Integer.class.getDeclaredMethod("parseInt", String.class); 
   ctx.registerFunction("parseInt", parseInt); 
   ctx.setVariable("parseInt2", parseInt); 
  ```

  > `registerFunction`和`setVariable`都可以注册自定义函数，但是两个方法的含义不一样，推荐使用`registerFunction`方法注册自定义函数。 

* Elvis运算符

   是三目运算符的特殊写法，可以避免null报错的情况 

  ```java
  name != null? name : "other"
  
  //简写为：
  name?:"other"
  ```

* 安全保证

  为了避免操作对象本身可能为null，取属性时报错，定义语法

  语法： `对象?.变量|方法` 

  ```unkown
  list?.length
  ```

* ...

> 参考
>
> * [SpEL表达式总结](https://www.jianshu.com/p/e0b50053b5d3) 比较全, 比较容易理解
>
> * [Spring Expression Language (SpEL)](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/core.html#expressions)



