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

> 包含了实现AOP时涉及的概念

* Aspect: A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes (the [schema-based approach](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)) or regular classes annotated with the `@Aspect` annotation (the [@AspectJ style](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)).
* Join point: A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
* Advice: Action taken by an aspect at a particular join point. Different types of advice include “around”, “before” and “after” advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point.
* Pointcut: A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.

------------

* Introduction: Declaring additional methods or fields on behalf of a type. Spring AOP lets you introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an `IsModified` interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
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

### proxy模式

运行时织入逻辑, 只能为方法织入逻辑

* 两种实现方案
  * JDK代理: 仅能够代理接口
  
  > 通过实现接口产生代理类实现
  
  * Cglib代理: 能代理接口和类
  
    > 通过继承类实现.
  
  > 好像... 默认代理接口时用的JDK代理, 代理类时用的Cglib代理
  
* 限制

  * 不能为非Spring工厂创建的Bean添加逻辑, 如自调用失效:

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

## 声明切点表达式

> 仅支持方法的连接点

# 参考

* [Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)

* [Introduction to Spring AOP](https://www.baeldung.com/spring-aop)
* [Spring AOP vs AspectJ](https://stackoverflow.com/questions/1606559/spring-aop-vs-aspectj)

