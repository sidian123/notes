# Task Execution and Scheduling

## TaskExecutor

* 介绍
  
* 含义与JDK的`Executor`基本一致
  
* 类型

  * `SyncTaskExecutor`

    任务在调用者线程中执行, 即同步的. 主要用于测试

  * `SimpleAsyncTaskExecutor`

    简单的为每个任务开启一个线程. 有并发上限, 达到上线时将阻塞调用者线程, 直到有线程结束.

  * `ThreadPoolTaskExecutor`

    适配了`ThreadPoolExecutor`, 并提供Bean属性用于配置.

  * `ConcurrentTaskExecutor`

    连接JDK `Executor`实例的一个适配类, 比较灵活性

* 使用? 

  可与普通类一样注入到容器中, 而`ThreadPoolTaskExecutor`提供了Bean属性, 可以直接在XML中配置.

## TaskScheduler

* 接口方法大致分类

  * `schedule()`

    仅在某个时间点运行且仅运行一次.

  * `scheduleAtFixedRate()`

    周期执行任务, 以两个任务开始的时间段为周期

  * `scheduleWithFixedDelay()`

    周期执行任务, 以上一个任务的结束与下一个任务的开始为周期, 即延迟一个时间段执行.

* `Trigger`接口

  ```java
  ScheduledFuture schedule(Runnable task, Trigger trigger);
  ```

  该方法中会用到, 极为灵活, 只要`Trigger`触发了并执行一次任务, 即可以触发多次.

  其中, 实现类`CronTrigger`使用cron表达式触发任务执行, 如

  ```java
  scheduler.schedule(task, new CronTrigger("0 15 9-17 * * MON-FRI"));
  ```

  > 每月的9到17号的日子中, 且位于周一至周五间内, 则在当天15:00点触发任务执行.

## 注解支持

Spring提供了注解来异步执行和调度任务.

### 使能与配置

#### 异步

* `@EnableAsync`使能任务异步执行

  默认使用的Bean, 以优先级降低的方式列出

    1. 若存实现了`AsyncConfigurer`的Bean, 则使用该Bean提供的执行器.
    2. 若容器中唯一存在`TaskExecutor`, 则使用该Bean
    3. 否则使用Bean名为`taskExecutor`的Bean
    4. 否则`SimpleAsyncTaskExecutor`将被使用
  
* `AsyncConfigurer`用于配置默认的执行器和默认的异常处理器, 如

  ```java
  @Configuration
  @EnableAsync
  public class AppConfig implements AsyncConfigurer {
  
      @Override
      public Executor getAsyncExecutor() {
          ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
          executor.setCorePoolSize(7);
          executor.setMaxPoolSize(42);
          executor.setQueueCapacity(11);
          executor.setThreadNamePrefix("MyExecutor-");
          executor.initialize();
          return executor;
      }
  
      @Override
      public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
          return new MyAsyncUncaughtExceptionHandler();
      }
  }
  ```
  
  > 若仅实现一个方法, 可继承`AsyncConfigurerSupport`

#### 调度

* `@EnableScheduling`使能任务调度

* `SchedulingConfigurer`用于配置

* 例子:

    ```java
    @Configuration
    @EnableScheduling
    public class AppConfig {
    }
    ```

### 使用

#### @Scheduled

* 原理: IOC调度

* 使用

  延迟5秒调用

  ```java
@Scheduled(fixedDelay=5000)
  public void doSomething() {
      // something that should execute periodically
  }
  ```
  
  固定5秒的频率调用

  ```java
@Scheduled(fixedRate=5000)
  public void doSomething() {
      // something that should execute periodically
  }
  ```
  
  添加初始延迟

  ```java
@Scheduled(initialDelay=1000, fixedRate=5000)
  public void doSomething() {
      // something that should execute periodically
  }
  ```
  
  使用cron表达式

  ```java
@Scheduled(cron="*/5 * * * * MON-FRI")
  public void doSomething() {
      // something that should execute on weekdays only
  }
  ```
  
  > 可使用`zone`指定cron表达式用于解析的时区.

* 注意点

  * 由于方法是被容器调用的, 因此返回值必须为`void`且无参
  * 不要在`@Configurable`类中使用`@Scheduled`注解, 会出问题的.

#### @Async

* 原理

  通过切面获取参数, 交与`TaskExecutor`执行

  且默认使用`proxy`模式, 因此, 相同的类中调用不会重复的织入

* 使用

  最简单的形式

  ```java
@Async
  void doSomething() {
      // this will be executed asynchronously
  }
  ```
  
  可以有参数, 因为该方法在代码中是被正常调用的

  ```java
@Async
  void doSomething(String s) {
      // this will be executed asynchronously
  }
  ```
  
  可以有返回值, 但由于是异步执行的, 因此只能返回`Future`类型的对象

  ```java
@Async
  Future<String> returnSomething(int i) {
      // this will be executed asynchronously
  }	
  ```
  
  > 除此之外, 还有`org.springframework.util.concurrent.ListenableFuture`和`java.util.concurrent.CompletableFuture`

  指定使用的执行器, 而不是`AsyncConfigurer`配置的默认执行器

  ```java
@Async("otherExecutor")
  void doSomething(String s) {
      // this will be executed asynchronously by "otherExecutor"
  }
  ```
  
* 异常处理

  如果有`Future`返回值, 当异步执行抛出异常时, `Future.get()`将抛出异常. 

  第二种方式, 在注册`AsyncConfigurer`时配置`AsyncUncaughtExceptionHandler`来处理所有异常

  ```java
  public class MyAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {
  
      @Override
      public void handleUncaughtException(Throwable ex, Method method, Object... params) {
          // handle exception
      }
  }
  ```

* 注意点

  `@Async`不能直接组合`@PostConstruct`来异步初始化Spring Bean, 而是需要其他Bean调用该Bean的异步初始化方法, 如

  ```java
  public class SampleBeanImpl implements SampleBean {
  
      @Async
      void doSomething() {
          // ...
      }
  
  }
  
  public class SampleBeanInitializer {
  
      private final SampleBean bean;
  
      public SampleBeanInitializer(SampleBean bean) {
          this.bean = bean;
      }
  
      @PostConstruct
      public void initialize() {
          bean.doSomething();
      }
  
  }
  ```

## 参考

[Task Execution and Scheduling](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#scheduling)

# Validation

* 未与Web层绑死, 可用在其他地方
*  JSR-303 Bean Validation API  被全面支持

> 是对Bean的校验, 参数本身校验貌似未支持, 这个可以靠基本类型验证, 暂时以后再了解;

> 参考
>
> * [Spring Validation](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#validation-beanvalidation)
> * [SpringMVC validation完成后端数据校验（较全面）](https://blog.csdn.net/m0_37589327/article/details/78648328)  : 写的还行吧

# Resource

`ResourceLoader`提供的`getResource`方法, 可以获取代表资源的`Resource`. 但资源是否真的存在, 需要`Resource.exists()`检测.

`ResourceLoader`会自动检查url是指向文件`file://C:/test.dat`还是classpath下`classpath:test.dat`

> 常用`ResourceLoader`的实现类`DefaultResourceLoader`

# 工具

## util

Spring的`util`包下, 含有各种工具类, 如

*  `Base64Utils` : A simple utility class for Base64 encoding and decoding. 
*  `StringUtils` : Miscellaneous [`String`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html?is-external=true) utility methods. 
*  `DigestUtils`: Miscellaneous methods for calculating digests.
*  `StopWatch`: 一个计算任务耗时的工具
* ...

> 参考[Package org.springframework.util](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/package-summary.html)

## beans

含与Bean相关的接口与类

*  [BeanUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/BeanUtils.html) : Static convenience methods for JavaBeans: for instantiating beans, checking bean property types, copying bean properties, etc. 
  * `copyProperties()`: bean间拷贝属性

> 参考[Package org.springframework.beans](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/package-summary.html)



# Cache

> 缓存逻辑位于`CacheAspectSupport.execute()`中

## 介绍

Spring提供了一个缓存抽象层, 规定了缓存注解的使用方法, 而抽象层的实现则由缓存提供者实现, 如Spring Data Redis提供了实现.

> 如果classpath下未提供任何实现, 则SpringBoot将使用`ConcurrentHashMap`作为缓存. 注: 不建议用在生成环境中.

缓存提供者主要实现` CacheManager `和` Cache `即可. `CacheManager`是缓存`Cache`的管理器, 拥有获取`Cache`的方法; `Cache`代表缓存, 拥有操作真正缓存的方法.

关于多线程, 虽说Spring有相关约定 ( 如`@Cacheable`的`sync`元素) ,该功能是否提供取决于缓存提供者.

关于使用, **必须**在配置类上添加注解启动缓存.

## 配置

### 启用缓存

配置类上添加` @EnableCaching `注解, 否则缓存注解将失效

### 配置缓存

需配置缓存提供者的`CacheManager`的实现类, SpringBoot在自动装配时, 提供了获取该类的回调, 如:

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

### 以Redis作为缓存

可通过配置`RedisCacheManager`来配置缓存, 但是推荐使用` RedisCacheConfiguration `, 更为方便.

可以配置的有:

* 过期时间, 默认永久

* 是否允许`null`值, 默认`yes`

* Redis的键是否使用前缀, 默认`yes`

* 默认的前缀, 默认`Cache`名

  > 如`@Cacheable(value = "blog",key = "#a")`, 并假设参数a为`1`, 则键为`blog::1`

* 键的序列化器, 默认`StringRedisSerializer`

* 值得序列化器, 默认`JdkSerializationRedisSerializer`

* 转化器?? 不认识...

下面的例子中配置了键值的序列化器:

```java
@EnableCaching
@Configuration
public class RedisConfiguration {
    @Bean
    public RedisCacheConfiguration redisCacheConfiguration(){
        return RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(RedisSerializer.string()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(RedisSerializer.string()));
    }
}
```

> 都使用`StringRedisSerializer`来序列化
>
> 这只是对Cache的配置, 不会影响到Spring Data Redis的`RedisTemplate`的配置.

## 使用

主要是关于注解的使用, SpEL的使用在注解中也很关键.

### @Cacheable

主要注解到方法上, 当方法被访问时, 如果被请求的资源已存在, 则直接返回缓存, 并不真正调用方法; 若请求的资源不存在, 则执行方法, 将结果缓存起来并返回.

注解的几个元素介绍:

* 键相关

    * `value`或`cacheNames` 指定`Cache`缓存名

    * `key` 指定键值, 默认由方法所有参数构造.

      > 注意, 在Redis中, 键是由`cacheNames`和`key`两个元素构造的

    * `keyGenerator` 自定义的键的构造器, 不能与`key`一同使用

  > 不指定`key`时的键的构造就是由`KeyGenerator`完成的

* 缓存管理&解析相关

    * `cacheManager` 自定义的缓存构造器, 不能同`cacheResolver`一同使用

    * `cacheResolver` 自定义的`cacheResolver`

* 缓存的条件相关

    默认方法调用的结果都会被缓存起来, 但也可配置是否缓存的条件, 条件有SpEL表达式给出

    * `condition` 设置是否缓存结果的条件, 方法执行前判断
    * `unless` 设置是否缓存结果的条件, 方法结束后判断, 因此表达式可引用结果对象.

* 同步

    * `sync`是否同步, 防止并发造成的多次操作缓存.

        > 该功能是否支持, 由缓存提供者决定

### SpEL环境

Spring提供了很多环境相关的元数据, 让使用者能在缓存注解中获取所有相关信息.

建议使用相关注解时, 仔细阅读文档.

所有环境元数据如下所示:

| Name          | Location           | Description                                                  | Example                                                      |
| :------------ | :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `methodName`  | Root object        | The name of the method being invoked                         | `#root.methodName`                                           |
| `method`      | Root object        | The method being invoked                                     | `#root.method.name`                                          |
| `target`      | Root object        | The target object being invoked                              | `#root.target`                                               |
| `targetClass` | Root object        | The class of the target being invoked                        | `#root.targetClass`                                          |
| `args`        | Root object        | The arguments (as array) used for invoking the target        | `#root.args[0]`                                              |
| `caches`      | Root object        | Collection of caches against which the current method is executed | `#root.caches[0].name`                                       |
| Argument name | Evaluation context | Name of any of the method arguments. If the names are not available (perhaps due to having no debug information), the argument names are also available under the `#a<#arg>` where `#arg` stands for the argument index (starting from `0`). | `#iban` or `#a0` (you can also use `#p0` or `#p<#arg>` notation as an alias). |
| `result`      | Evaluation context | The result of the method call (the value to be cached). Only available in `unless` expressions, `cache put` expressions (to compute the `key`), or `cache evict` expressions (when `beforeInvocation` is `false`). For supported wrappers (such as `Optional`), `#result` refers to the actual object, not the wrapper. | `#result`                                                    |

### 其他注解

由于注解元素的使用方法与`@Cacheable`差不多, 便不详细介绍了.

* `@CachePut` 方法调用后, 结果都会被缓存, 并不会干涉方法的执行

* `@CacheEvict`方法被调用后, 缓存被清空. 额外元素如下

  * `allEntries` 是否`Cache`的所有键值都被清空, 默认`false`. 不能与`key`同时存在

    > 否则仅清空一个键

  * `beforeInvocation` 是否在方法调用前清空缓存, 默认`false`

    > 清空缓存将不被方法所干涉, 如方法是否抛出异常.

* `@Caching` 让同类型的多个注解, 标注在同一方法上, 如

  ```java
  @Caching(evict = { @CacheEvict("primary"), @CacheEvict(cacheNames="secondary", key="#p0") })
  public Book importBooks(String deposit, Date date)
  ```

* `@CacheConfig`注解在类上, 为其他注解设置默认配置

## 日!天坑!!

![image-20200208183602564](.Spring%20Core/image-20200208183602564.png)

大部分初始化回调 ( 如`@PostConstruct`, `BeanPostProcessor `等 ) 都是在第一个`for`循环中执行的, 而Spring Cache的初始化是在第二个`for`循环中执行的.

因此, 初始化的缓存注解都是**失效的**!!!

## 参考

* [Cache Abstraction](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/integration.html#cache)

* [Spring Boot Cache](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching)

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

# Email

* 目的: 该模块主要用于屏蔽底层实现, 简化邮箱的使用.

* 首先需要引入JavaMail

  ```xml
  <dependency>
      <groupId>com.sun.mail</groupId>
      <artifactId>javax.mail</artifactId>
      <version>1.6.2</version>
  </dependency>
  ```

* 核心组件

  * `MailServer`接口代表服务器, 用于发送邮件
    * `JavaMailSenderImpl`其实现类
  * `MailMessage`接口表示一封邮件
    * ` SimpleMailMessage `一封简单的文本邮件
    * `MimeMailMessage`一封MIME类型的邮件

* 使用

  * 方式一: 使用`MailMessage`创建一封邮箱, 让`MailSender`发送
  * 方式二: Spring提供了`MimeMessageHelper`类帮助构建邮件, 然后再让`MailServer`发送

  * 方式三: 通过`MimeMessagePreparator`暴露底层类, 来装配邮件, 然后`MailServer`发送

> 参考[Email](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#mail)

# 其他

## Spring boot 启动

### SpringApplication

用于在Java主方法中启动Spring应用, 将执行如下步骤:

- Create an appropriate `ApplicationContext` instance (depending on your classpath)
- Register a `CommandLinePropertySource` to expose command line arguments as Spring properties
- Refresh the application context, loading all singleton beans
- Trigger any [`CommandLineRunner`](http://localhost:63342/Blog/spring-boot-2.2.1.RELEASE-javadoc.jar/org/springframework/boot/CommandLineRunner.html) beans

常用`run()`启动应用

```java
@Configuration
@EnableAutoConfiguration
public class MyApplication  {

    // ... Bean definitions

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

或提供更多的配置

```java
public static void main(String[] args) {
    SpringApplication application = new SpringApplication(MyApplication.class);
    // ... customize application settings here
    application.run(args)
}
```

数据源来源三种

- The fully qualified class name to be loaded by `AnnotatedBeanDefinitionReader`

  > 即`Configuration`注解标注的类

- The location of an XML resource to be loaded by `XmlBeanDefinitionReader`, or a groovy script to be loaded by `GroovyBeanDefinitionReader`

- The name of a package to be scanned by `ClassPathBeanDefinitionScanner`

  > 即动态扫描的Bean.

###  CommandLineRunner 

* 当容器初始化完毕后, 执行该接口的方法, 用于处理命令行参数
* 仅在`SpringApplication`初始化的容器中使用
* `ApplicationArguments`与`CommandLineRunner`类似, 但提供了更便捷的方式获取命令行参数

