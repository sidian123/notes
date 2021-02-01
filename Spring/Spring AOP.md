# 一 引言

在面向对象编程OOP中, 业务对象可能不止业务逻辑代码, 还会有非业务逻辑代码, 如日志; 而面向切面编程AOP, 作为OOP的补充, 能够将非业务逻辑代码抽离出来, 增加了代码的模块化的同时让Coder的关注点更多的聚焦在业务代码上.

> Spring对AOP的介绍
>
>  Aspect-oriented Programming (AOP) complements Object-oriented Programming (OOP) by providing another way of thinking about program structure. The key unit of modularity in OOP is the class, whereas in AOP the unit of modularity is the aspect. Aspects enable the modularization of concerns (such as transaction management) that cut across multiple types and objects. (Such concerns are often termed “crosscutting” concerns in AOP literature.) 

# 二 概念

## 我的介绍

* 抽象描述

  在**业务逻辑执行序列**中, 有很多**连接点**JoinPoint, 都可以插入**通知**Advice, 通知在那个接入点执行, 由**切点**表达式Pointcut Expression给出. 将所有连接点按照功能分模块后, 一个模块的所有连接点则组成了**切面**Aspect

* 通俗描述

  在Spring AOP中, **连接点**JoinPoint就是方法执行前或后的那一刻, 而**通知**Advise则是一个要在连接点执行的方法或动作, **切点**Pointcut是一个匹配表达式, 即通知与连接点的匹配规则. 最后, 相关联的一组通知, 放在同一个类中, 这个类就表示**切面**Aspect

## Spring的介绍

* Aspect: A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes (the [schema-based approach](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)) or regular classes annotated with the `@Aspect` annotation (the [@AspectJ style](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)).
* Join point: A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
* Advice: Action taken by an aspect at a particular join point. Different types of advice include “around”, “before” and “after” advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point.
* Pointcut: A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.

* Introduction: Declaring additional methods or fields on behalf of a type. Spring AOP lets you introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an `IsModified` interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)

-----------

> 包含了实现AOP时涉及的概念

* Target object: An object being advised by one or more aspects. Also referred to as the “advised object”. Since Spring AOP is implemented by using runtime proxies, this object is always a proxied object.
* AOP proxy: An object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy is a JDK dynamic proxy or a CGLIB proxy.
* Weaving: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.

## Advice类型

* Before advice: Advice that runs before a join point but that does not have the ability to prevent execution flow proceeding to the join point (unless it throws an exception).
* After returning advice: Advice to be run after a join point completes normally (for example, if a method returns without throwing an exception).
* After throwing advice: Advice to be executed if a method exits by throwing an exception.
* After (finally) advice: Advice to be executed regardless of the means by which a join point exits (normal or exceptional return).
* Around advice: Advice that surrounds a join point such as a method invocation. This is the most powerful kind of advice. Around advice can perform custom behavior before and after the method invocation. It is also responsible for choosing whether to proceed to the join point or to shortcut the advised method execution by returning its own return value or throwing an exception.

## 代理模式

Spring AOP的实现是基于代理的

### Proxy模式

运行时织入逻辑, 只能为方法织入逻辑

* 两种实现方案
  * JDK代理: 仅能够代理接口
  
    > 通过实现接口产生代理类实现
  
  * Cglib代理: 能代理接口和类
  
    > 通过继承类实现.
  
  > 好像... 默认代理接口时用的JDK代理, 代理类时用的Cglib代理
  
* 限制

  * 使用非Spring容器创建的实例引用调用, 将无AOP逻辑, 如自调用失效:

    ```java
    @Transactional
    class A{
        public void a(){}
        public void b(){
            a();
        }
    }
    ```

    > `a()`的调用无AOP逻辑

    解决方案: `a()`是通过`this`调用的, `this`指向的是真实对象; 可通过Spring注入自身实例, 这是一个代理类的实例, 拥有AOP逻辑, 如

    ```java
    @Transactional
    class A{
        @Autowired
        private A instance;
        public void a(){}
        public void b(){
            instance.a();
        }
    }
    ```

  * 最好注解在`public`方法上：非`public`方法, `static`方法容易出现注解失效的问题。
  
    > 貌似只要非`private`方法就行了?

### AspectJ 模式

需要额外编译器, 编译时织入逻辑, 可以在任何地方织入切面逻辑

# 三 使用

Spring提供了`@AspectJ `注解方式和XML配置方式, 这里仅介绍注解方式.

Spring使用了AspectJ5相同的注解, 和AspectJ切点表达式的实现库, 但是AOP运行时仍由Spring提供.

## 启动@AspectJ支持

需在`@Configuration`类上添加`@EnableAspectJAutoProxy`注解

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

启动注解后, 被织入的Bean将被自动代理.

> 确保` aspectjweaver.jar `在classpath下存在
>
> ```xml
> <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
> <dependency>
>     <groupId>org.aspectj</groupId>
>     <artifactId>aspectjweaver</artifactId>
>     <version>1.9.4</version>
> </dependency>
> 
> ```

## 声明切面

被`@Aspect`标注的Bean将被Spring自动检测到, 并配置AOP.

```java
@Aspect
@component
public class NotVeryUsefulAspect {

}
```

> 注意, 切面自己不能被其他切面织入.

## 声明切点

Spring AOP使用切面语言AspectJ的切点表达式语法, 见  [AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/released/progguide/index.html) . 但是却有些差别, 即Spring AOP仅支持方法级的连接点, 且不是全部支持AspectJ的切点表达式语法.

> 由于AspectJ语法复杂, 骚操作多, 这里仅给出通用的, 没必要全都掌握.

* **切点声明**由两部分组成: **切点表达式**和**切点方法原型**, 如

    ```java
    @Pointcut("execution(* transfer(..))") // 切点表达式
    private void anyOldTransfer() {} // 切点方法原型
    ```

    > * 方法必须返回`viod`
    > * 方法修饰符任意, 可被继承, 切点声明供子类使用
    >
    > * Proxy代理模式下, 切点表达式指向的方法必须是`public`

* **切点表达式**分多种, 由其**指示符**Designator决定类型, 加上对应内容, 来限制方法匹配范围.

* 切点表达式之间可以通过逻辑运算符`&&`,`||`,`!`来组合

### 指示符

> 并不完全支持AspectJ的所有指示符

下面都是限制方法(或连接点)匹配的方式, 只是从**不同方面**来限制方法匹配.

#### exection

`execution`: 通过匹配方法签名来限制匹配, 最最最常用.

语法规则

```
execution(<修饰符>? <返回类型> <类型全限定名>?<方法名>(<参数列表>) <抛出异常列表>?)
```

* 可选部分: 修饰符, 类型全限定名, 抛出异常列表

* 对于修饰符, Proxy代理模式, 只能代理`public`方法, 即使编写表达式未指定. <span style="color:red">注意好了!下面例子中不在强调</span>

* 返回值中, `*`表示任意类型

* 方法名中, 可使用通配符`*`

* 参数列表中
  * `()`无参数
  * `(*)`仅一个参数, 且任意
  * `(*,String)`两个参数, first任意, second必须`String`类型
  * `(..)`任意数量任意类型
  
* 关于类型全限定名, 有点坑, 还是直接看下面例子吧

  > 但有一点, 如
  >
  > ```java
  > execution(* ..Sample+.sampleGenericMethod(*)
  > ```
  >
  > 里面的`+`表示类型必须是`Sample`或其子类

-----------

例子

* 所有公共方法

  ```java
  execution(public * *(..))
  ```

* 方法名以`set`开始的方法

  ```java
  execution(* set*(..))
  ```

* 指定类型的所有方法

  ```java
  execution(* com.xyz.service.AccountService.*(..))
  ```

* 包下所有直接类的所有方法

  ```java
  execution(* com.xyz.service..(..))
  ```

* 包及其子包下所有类的所有方法

  ```java
  execution(* com.xyz.service...(..))
  ```

#### 其他

- `within`: 限制方法所属类型的全限定名. 

  > 匹配颗粒度要比`execution`大

  * 方法要位于`service`包任意类型下

    ```java
    within(com.xyz.service.*)
    ```

  * 方法位于`service`包及其子包的任意类型下

    ```java
    within(com.xyz.service..*)
    ```

- `this`: 匹配对应类型的代理对象

  匹配`AccountService`类型的代理对象

  ```java
  this(com.xyz.service.AccountService)
  ```

- `target`: 匹配对应类型被代理的对象.

  > 就是A代理B, 则B是target, A是B的proxy

  匹配`AccountService`的被代理对象

  ```java
  target(com.xyz.service.AccountService)
  ```

- `args`: 通过方法参数来限制匹配

  仅一个参数, 且为` Serializable `类型

  ```java
  args(java.io.Serializable)
  ```

  > 与` execution(* *(java.io.Serializable)) `相比, `arg`是运行时判断参数实际类型, 而`execution`则比对声明的方法签名.

  > 其他例子见3.4.6小节

- `@args`: 通过运行时参数上的注解类型来限制匹配

  匹配一个参数, 且被`@Classified`标注的方法

  ```java
  @args(com.xyz.security.Classified)
  ```

- `@target`, `@within`, `@annotation`: 通过方法所处类上的注解类型来限制匹配

  匹配方法所属类存在`@Transactional`注解的所有方法

  ```java
  @target(org.springframework.transaction.annotation.Transactional)
  ```

  ```java
  @within(org.springframework.transaction.annotation.Transactional)
  ```

  ```
  @annotation(org.springframework.transaction.annotation.Transactional)
  ```

- `bean`通过Bean的名字来限定, 支持通配符`*`

  > 该指示符由Spring提供
  
  * 匹配名叫`tradeService`的Bean
  
    ```java
    bean(tradeService)
    ```
  
  * 用通配符`*`来匹配Bean名
  
    ```java
    bean(*Service)
    ```
  
    

### 表达式组合

通过逻辑操作符组合, 操作符有`&&`,`||`,`!` , 操作符可以是**切点表达式**, 或者切点的**方法调用**

```java
@Pointcut("execution(public * (..))")
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.someapp.trading..)")
private void inTrading() {} 
//通过&&组合, 且使用方法调用的形式
@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {} 
```

## 声明通知

切点表达式匹配到方法后, 需要指定对应类型的通知, 如方法前执行的通知, 方法后通知, 环绕通知等等

需要给通知切点信息, 有两种方法

* 指定切点的方法调用

  ```java
  @Aspect
  public class BeforeExample {
    //注解值是切点的方法调用
    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
      public void doAccessCheck() {
          // ...
      }
  }
  ```

* 直接内嵌切点表达式

  ```java
  @Aspect
  public class BeforeExample {
  
      @Before("execution(* com.xyz.myapp.dao..(..))")
      public void doAccessCheck() {
          // ...
      }
  
  }
  ```

> 两种方式各有优点, 如第一种, 切点中可通过组合的方法复用已有的切点表达式; 第二种, 方便快捷...

下面介绍不同类型的通知

### Before通知

方法前执行

```java
    @Before("execution(* com.xyz.myapp.dao..(..))")
    public void doAccessCheck() {
        // ...
    }
```

### After Returning通知

方法正常返回后执行

```java
  @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
```

获取方法返回值

```java
    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
```

注意:

* `returning`属性值必须与方法参数名一致
* Spring AOP会从参数上获取类型信息, 并缩小方法的匹配范围

### After Throwing通知

方法抛出异常后执行

```java
  @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }
```

获取抛出的异常

```java
@AfterThrowing(
    pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
    throwing="ex")
public void doRecoveryActions(DataAccessException ex) {
    // ...
}
```

注意

1. `throwing`属性值必须与参数名一致

2. Spring AOP同样会从参数上获取类型信息, 并缩小方法的匹配范围.

   > 若匹配所有异常, 请使用` Throwable `类型的参数

### After (Finally)通知

方法返回后, 无论是否正常返回, 都执行. 主要用于释放资源.

```java
  @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }
```

### Around通知

能够同时在方法前后执行, 能够控制是否真的执行方法

```java
  @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }
```

注意

* 必须有`ProceedingJoinPoint`参数, 使用该类的`proceed()`方法可手动执行目标方法, 或不执行, 或执行多次. 它的`proceed(Object[])`方法可传入参数, 覆盖原来的.
* Around通知的返回值, 将作为目标方法调用的返回值.

### 通知的参数

上述例子中, 介绍了部分通知如何获取参数的方案, 这里详细讲解.

#### 获取JoinPoint

任何通知的方法都可以在第一个参数中声明` JoinPoint `获取该方法的一些信息.

> Around通知中, 强制存在` ProceedingJoinPoint `参数, `JoinPoint`的子类

`JoinPoint`中有用的方法:

- `getArgs()`: Returns the method arguments.
- `getThis()`: Returns the proxy object.
- `getTarget()`: Returns the target object.
- `getSignature()`: Returns a description of the method that is being advised.
- `toString()`: Prints a useful description of the method being advised.

#### 获取方法参数

通过组合命令与`args`即可获取参数值

```java
@Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```

注意 

* `args(account,..) `表示匹配的方法有任意多个参数, 第一个必须是`Account`类型的.

* 通知的参数名必须与`args`中一致, 才会获取到参数值.

  > `args`中参数名不必与目录方法参数名一致, 只要位置对应即可.

------

上述例子的另一种写法

```java
@Pointcut("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```

#### 获取注解

首先注解定义

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```

然后通知中获取注解

```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```

#### 获取与泛型相关的参数

首先将被代理的接口

```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```

匹配接口的第一个方法, 并获取参数

```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```

> 注意,  匹配的方法的参数必须是`MyType`类型, 即匹配已实例化的泛型实参

但是上述方法对集合无效, 只能如下获取`Collection<?>`类型的参数

```java
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<?> param) {
    // Advice implementation
}
```

> 在消息体内需手动检查泛型类型

### 通知的顺序

不同切面的通知在同一个连接点运行时, 那个通知先运行? 方法(连接点) 执行前, 高优先级切面的通知先执行, 方法执行后, 高优先级切面的通知后执行.

切面通过继承` Ordered `接口, 或标注`@Order `注解可设置切面的优先级, 数值越低的优先级越高.

> 貌似无指定时, 优先级为0. 涉及到此类问题时建议还是手动设置以下优先级
>
> 经调试发现, 事务,缓存注解默认都是最低的优先级, 即2^32^-1

同一切面在同一连接点运行时, 那个优先? 这是未知的, 因为Spring无法获取方法在源码中定义的先后顺序

## Introductions

 就是, 在**目标类型匹配表达式** (与上面说的切点表示微微有点不同) 匹配到的类型中, 插入一个接口的实现类的所有接口成员. 主要用于统计信息

```java
@Aspect
public class UsageTracking {
    //value指定匹配类型的表达式, defaultImpl给出要插入接口成员的实现类
  @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;
   //现在可通过接口找到代理类, 并添加而外逻辑	
    @Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        //统计该节点被调用的次数
        usageTracked.incrementUseCount();
    }

}
```

> 详细见注释
>
> 不知道可不可以插入无接口的类, 未测试.

## Demo

* 切被注解标注的方法, 切被注解标注的类的所有公有方法.

  ```java
  @Aspect
  public class RoleAspect {
  
      @Around("@annotation(role) || (@within(role) && execution(public * *(..)))")
      public Object around(ProceedingJoinPoint pjp, Role role) throws Throwable {
          // 提取角色限制
          List<Integer> roles = extractRoles(pjp, role);
          // 鉴权
          MyAssert.isTrue(authorize(roles, TokenHolder.getToken()), CODE_401.toStatus());
          return pjp.proceed();
      }
  }
  ```

# 四 其他

## 关于参数名获取

不写了, 见[ Determining Argument Names ](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj-advice-params-names)

## 解决循环引用

注入的对象使用懒注入`@Lazy`

# 参考

* [Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)
* [Introduction to Spring AOP](https://www.baeldung.com/spring-aop)
* [Spring AOP vs AspectJ](https://stackoverflow.com/questions/1606559/spring-aop-vs-aspectj)
* [The AspectJTM Programming Guide](https://www.eclipse.org/aspectj/doc/released/progguide/index.html)

