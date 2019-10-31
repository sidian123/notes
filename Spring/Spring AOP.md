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

        > a()的调用无aop逻辑

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

> 参考[Spring AOP vs AspectJ](https://stackoverflow.com/questions/1606559/spring-aop-vs-aspectj)

