* 介绍

  在面向对象编程OOP中, 业务对象可能不止业务逻辑代码, 还会有非业务逻辑代码, 如日志; 而面向切面编程AOP, 作为OOP的补充, 能够将非业务逻辑代码抽离出来, 增加了代码的模块化的同时让Coder的关注点更多的聚焦在业务代码上.

* 概念

  * 抽象描述

    在**业务逻辑执行序列**中, 有很多**连接点**JoinPoint, 都可以插入**通知**Advice, 通知在那个接入点执行, 由**切点**表达式Pointcut Expression给出. 将所有连接点按照功能分模块后, 一个模块的所有连接点则组成了**切面**Aspect

  * 通俗描述

    在Spring AOP中, **连接点**JoinPoint就是方法执行前或后的那一刻, 而**通知**Advise则是一个要在连接点执行的方法或动作, **切点**Pointcut是一个匹配表达式, 即通知与连接点的匹配规则. 最后, 相关联的一组通知, 放在同一个类中, 这个类就表示**切面**Aspect

  * 总结

    * 一幅不太给力的图表示上述过程

       ![img](.Spring%20AOP/Program_Execution-300x180.jpg) 

    * 总之, 就是一个在代码执行流中**插入**和**如何插入**其他代码的过程, 只是被高度抽象化罢了...

* 

* 两种实现模式

  * `proxy`模式: 一种代理方案, 运行时织入逻辑, 只能为方法织入逻辑

    * 两种实现方案
      * JDK代理
      * Cglib代理（默认）

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

      * 最好注解在public方法上：非public方法, static方法容易出现注解失效的问题。

    * `AspectJ `模式: 需要额外编译器, 编译时织入逻辑, 可以在任何地方织入切面逻辑

* 参考
  * [Introduction to Spring AOP](https://www.baeldung.com/spring-aop)
  * [Spring AOP vs AspectJ](https://stackoverflow.com/questions/1606559/spring-aop-vs-aspectj)

