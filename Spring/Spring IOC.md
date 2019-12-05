[TOC]

# 一、介绍

Spring Framework是Ioc容器的实现，ioc容器根据**元数据配置**来**实例化、装配**（根据依赖关系）Bean，甚至管理Bean的**生命周期**（prototype的Bean实例化配置完整后的生命周期不归容器管理）。元数据配置定义了Bean的定义和依赖关系。bean与类相似，是一个模板，用来产生对象，每个bean都有自己的生命周期。一个类可以对应多个Bean。

spring容器实现了IOC（控制反转）和DI（依赖注入）的功能，因此很有必要理解这两个名词：[控制反转 vs 依赖注入](https://blog.csdn.net/jdbdh/article/details/82800306)

ioc是通过元数据配置初始化容器的，容器的实现有很多种，提供元数据配置的方式也有很多种。下面先讲元数据的提供方式。

- 基于xml的配置：元数据可以通过xml配置文件提供，由`<bean/>`定义Bean、其他bean的依赖关系和声明周期等内容。
- 基于注解的配置：通过`@Componet`等注解可以将类作为Bean。
- 基于java的配置：元数据通过配置类提供，常使用`@Configuration`、`@Bean`等注解定义Bean

`BeanFactory`接口提供了容器基本的配置框架和功能，但是`ApplicationContext`接口提供了更多的企业级功能，是`BeanFactory`的超集，一般使用`ApplicationContext`接口的实现作为容器。而`WebApplicationContext`接口适用于web应用。

对于基于xml的配置，通常将xml文件传给`ApplicationContext`的实现类`ClassPathXmlApplicationContext`来产生IOC容器。由于基于注解的配置和基于java的配置都用到了注解的方法，因此传入`ApplicationContext`的实现类`AnnotationConfigApplicationContext`来产生IOC容器。

容器的功能是可以扩展的，除了实现`ApplicationContext`接口扩展功能，也可以通过实现`BeanPostProcessor`来扩展功能，比如注解配置和AOP代理功能都是实现`BeanPostProcessor`接口来扩展的。

# [二、容器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-basics)

**元数据配置**：**Bean的定义**和**依赖关系**是通过元数据配置（*configuration metadata* ）来描述的，*configuration metadata* 可以通过xml、java注解或java代码来呈现。不管是xml配置、注解配置还是java代码配置，都是可以混合嵌套使用的。

一般使用ApplicationContext的实现类来创建容器，比如ClassPathXmlApplicationContext类可以从classpath路径下加载元数据配置；FileSystemXmlApplicationContext可以文件系统路径来获得元数据配置。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```



可以传入一个或多个配置文件，也可以在传入一个配置文件，但是这个配置文件中导入了其他配置文件，通过import标签来实现，resource指定的地址是相对于使用import标签的配置文件而言的。

```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```



可以通过getBean方法使用得到bean的实例

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```



但是spring文档并不建议这样使用，因为这会导致spring api侵入了你的程序，而是建议通过使用依赖注入功能，比如注解@Autowired。

# [三、Bean概述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition)

一个ioc容器管理多个Bean，而Bean是根据元数据配置来创建的。在容器中，Bean定义被表示为`*BeanDefinition*` 对象，这些对象包含了类的全限定类名、Bean在容器中的行为（scope、生命周期、callbacks等等）、依赖和其他配置。元数据配置会被转化为`*BeanDefinition*` 的一些列属性值，如下表所示：

*The bean definition:*

| Property                 | Explained in…                                                |
| ------------------------ | ------------------------------------------------------------ |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) |

## Bean命名

每个Bean都可以有多个标识符，且标识符在容器中是唯一的。

> 同一bean之间的标识符互为别名

在基于xml的配置中，可以使用`Bean`标签的`id`和`name`属性来指定标识符。

* `id`只能指定一个标识符，`name`可以指定多个标识符，多个标识符通过`，`,`;`或空格分隔。
* 如果没有提供`name`和`id`值，那么容器会自己为bean产生一个未知的, 唯一的标识符.
* 如果使用`@Component`注解Bean，且不指定名字，那么会将类名的第一个字母转化为小写，然后作为Bean的标识符。
* 如果使用@Bean注解方法，会使用方法名作为Bean的标识符。

### Bean定义外设置别名

在`Bean`元素定义的同时可以直接定义多个标识符，互为别名; 在Bean定义之外也可以为Bean定义别名。

XML配置:

```
<alias name="fromName" alias="toName"/>
```

其中，`name`为一个Bean的标识符，`alias`指定别名。

## 实例化Beans

Bean定义就是一个模板，根据该Bean的定义来产生一个或多个对象。需要指定类的全限定名class，如果该类是静态内部类，则必须使用内部类的二进制名，也就是：外部类全限定名+$+内部类名。下面的三个构造方法如果需要注入依赖，都可以通过子标签constructor-arg来注入。

### 通过构造函数实例化

直接使用如下形式，会直接调用无参构造函数，如果需要通过构造函数注入依赖。

```
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```



### 通过静态工厂方法实例化

需要给静态工厂返回的对象配置一个Bean定义，但class属性不是返回值对应的类。

有一个类，类中含有静态工厂方法：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```



通过如下方法实例化：

```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```



class指定含有静态工厂的类，factory-method指定静态工厂方法，不用给出返回类型，因为通过反射可以得到。id为静态工厂返回的对象的标识符，不是class的标识符。注意，scope默认为singleton，因此该方法只会调用一次，维护一个单实例。注意，class指定的类一般不会实例化，因为它不是bean。

### 通过实例工厂实例化

同样需要给工厂方法返回的对象配置一个Bean定义，但是不需要指定class属性。

java类如下：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```



配置文件：

```
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```



其中，factory-bean指定含有工厂方法的类的标识符，factory-method指定工厂方法，bean可由反射得知。上面三个bean都会被实例化。

### [实现FactoryBean创建工厂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension-factorybean)

同时也可以通过实现FactoryBean来创建Bean。实现FactoryBean的是工厂类，该接口的getObject方法返回的是被生产的Bean，isSingleton方法当该Bean为单实例时返回true，getObjectType方法返回bean的类型。

举一个mybatis和spring的中间件的工厂bean：

```
    //SqlSessionFactory工厂
	<bean id="SqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:/mybatis-config.xml"/>
	</bean>
```



由于SqlSessionFactoryBean实现了FactoryBean，因此该Bean是一个工厂。如果查看该类对FactoryBean的实现会发现，该工厂是用来生产SqlSessionFactory的，且为单实例。因此，通过id的值SqlSessionFactory可以获得该类（SqlSessionFactory）的对象。如果要获得工厂本身而不是生产的Bean，则需要在id前加个前缀“&”，即可获得工厂Bean。

# [四 依赖](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-dependencies)

## 依赖注入

依赖注入（DI）是一个过程，在这个过程中，对象定义它的依赖，这个依赖通过构造函数参数、给工厂方法的参数、setter方法参数（在实例化后）来注入的。DI可分为两类：基于构造函数的依赖注入和基于setter方法的依赖注入。其中工厂方法的注入和构造函数的依赖注入类似。基于setter方法的注入在Bean实例化后进行。两种注入方法可以一起使用。

一般情况下，使用构造函数注入**必需**的依赖，使用setter方法注入**可选**的依赖。

### 基于构造方法的依赖注入

就是通过构造函数的参数传递依赖。在xml配置中，通过`<constructor-arg/>` 指定依赖。下面有四种注入方式，前两种不推荐。

**参数顺序**

第一个constructor-arg指定的依赖传给第一个参数，第二个依赖传给第二个参数。文档中说如果依赖之间没有继承关系，就没有模糊性存在，所以工作的好。。但是我测试了下，貌似没啥关系。

```
<beans>
    <bean id="thingOne" class="x.y.ThingOne">
        <constructor-arg ref="thingTwo"/>
        <constructor-arg ref="thingThree"/>
    </bean>

    <bean id="thingTwo" class="x.y.ThingTwo"/>

    <bean id="thingThree" class="x.y.ThingThree"/>
</beans>
```



**参数类型匹配**

在constructor-arg中指定类型type就行了

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```



**参数索引**

通过index指定为哪个参数的依赖，index从0开始。

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```



**参数名字**

通过name指定参数名

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```



最后文档说到，要使用参数名匹配参数，需要编译时设置debug标志，然后spring才能够查看参数名。我在eclipse中没有使用debug模式运行时也能成功运行。不知是不是我理解错了。

### 基于setter的依赖注入

在Bean实例化后，会调用setter进行依赖注入。通过<property>标签来指定依赖。

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>
```



### 依赖解析过程

通过元数据配置，将创建和初始化`ApplicationContext`容器. 加载配置时，会检测引用是否指向了不存在的bean和是否存在**循环依赖**。

1. 实例化

   由于默认Bean为`singleton`作用域和预初始化的，因此首先实例化所有的Bean。

2. 属性装配

   实例化后, Spring会尽可能慢地解析, 和注入依赖

   > 这样能保证, 在没有循环依赖的情况下能够正常注入依赖，不会出现要注入依赖时，依赖对象还没有实例化的情况

   > 在注入依赖时，依赖自身必须被完全装配好, 即在`BeanPostProcessor`的方法回调执行完之后才算装配结束. 详细见第六章-Bean生命周期

>循环依赖
>
>举个例子，Class A的实例化（通过构造函数）需要Class B的实例，Class B的实例化需要Class A的实例，因此相互之间需要对方的存在，导致创建失败，抛出`BeanCurrentlyInCreationException`异常

> 如果Bean不预初始化，那么运行程序很久之后，初次使用了该bean，但是发现丢失或无效的属性，那么抛出异常。因此spring才会默认Bean预初始化，因为在容器加载元数据配置时就可以找出错误。

## 详细依赖配置

4.1节中讲到了依赖注入的两种方法和注入过程。依赖通过标签`<constructor-arg/>`或`<property/>`的属性`value`或`ref`指定。如果依赖是简单类型，则可以使用value直接赋值；如果依赖是Bean对象，则通过`ref`指定bean。除了使用**value和ref属性**指定依赖，还可以使用**子标签**来给出依赖。

### 简单值（基本类型、字符串等等）

简单的值可以通过value属性给出：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```



或者使用`<value/>`子标签

```xml
<bean id="..." class="...">
    <property name="...">
            <value>some value here</value>
    </property>
</bean>
```

`<value/>`标签有一个非常好的功能，就是可以为`java.util.Properties`对象赋值：

```xml
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

如果依赖为其他Bean的标识符，则可以使用`<idref/>`标签，该标签可以在容器加载阶段验证`id`所属的Bean是否存在（在scope不是sintleton时很有用），下面的相当于 `<property name="targetName" value="theTargetBean"/>`

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

### 对Bean的引用

如果依赖为其他的Bean，则需要使用`ref`属性指定依赖。同时可以使用`<ref/>`标签指定依赖，这样在容器加载时会验证该Bean是否存在（即使不是singleton的作用域）。

通过Bean属性指定在**相同容器或父容器**的的Bean，无论是否在相同xml文件中：

```xml
<ref bean="someBean"/>
```

通过`parent`属性指定**父容器**中的Bean：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

### 内部Bean

如果一个Bean只作为一个且仅一个Bean的依赖，此时可以直接将该Bean定义为内部Bean。内部Bean的`id`会被忽略，不用给出。内部Bean的scope对Bean的创建没有作用，内部Bean会随着外部Bean创建而创建。内部bean创建时会调用他的生命周期方法。scope作用域会影响内部bean销毁函数的调用。这是非常少见的使用场景。。

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

### 集合

如果依赖是集合，则可以使用子标签`<list/>`, `<set/>`, `<map/>`, and `<props/>`设置集合的属性，分别对应java集合类型：`List`, `Set`, `Map`, and `Properties`

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map的键值或set的值可以是如下元素（标签）：

```
bean | ref | idref | list | set | map | props | value | null
```

**集合合并**

如果Bean继承父Bean，那么子Bean中注入集合依赖可以**合并**和**覆盖**父Bean依赖的集合，需要合并的属性要设置`merge="true"`

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

下面子Bean的adminEmails属性的内容如下：

> ```properties
> administrator=administrator@example.com
> sales=sales@example.com
> support=support@example.co.uk
> ```
>

上面的所有集合都可以合并，但是`<list/>`特殊点，因为list是有序的，因此，后面子Bean的集合会插入到父Bean集合的后面。

**强类型集合**

java 5之后出现了泛型，因此类中使用了泛型，容器会根据这个类型将依赖转化为合适的类型注入：

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

容器会将`key`当做`String`，`value`转化为`Float`类型。

### null和空字符串

注入空字符串

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

注入`null`

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

### 其他

一些没多大用处的，只是便于使用的内容，比如使用p-namespace和c-namespace来简化xml配置，什么复杂的属性名注入，学多了头疼。

> 参考https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-p-namespace

## depends-on

depends-on强制一个或多个bean先于使用了depends-on属性的Bean初始化，无论是否需要depends-on指定的bean作为依赖注入。在下面的例子中，就算test2为原型作用域或者lazy-initialized，也要先于test初始化。

```
	<bean id="test" class="com.luo.main.Test" depends-on="test2" ></bean>
	<bean id="test2" class="com.luo.main.Test2" scope="prototype"></bean>
```



在Bean销毁时，Bean也要优先于deponds-on的Beans，**但是仅限于为singleton的Bean！！千万注意。**

## 懒初始化

默认Bean是singleton单实例和预加载的，但是lazy-init属性可以实现懒加载

```
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
```



在<beans/>标签中使用default-lazy-init可以设置**该元素**中的bean默认为懒加载

```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```



## 自动装配

如果Bean和依赖关系太多了，手动注入依赖太麻烦了，因此spring提供自动装配的功能。在基于xml的配置中，可以通过设置<bean/>元素的autowire属性来设置自动装配模式。下面列出了四种装配模式：

| Mode          | Explanation                                                  |
| ------------- | ------------------------------------------------------------ |
| `no`          | (Default) No autowiring. **Bean references must be defined by ref elements**. Changing the default setting is not recommended for larger deployments, because specifying collaborators explicitly gives greater control and clarity. *To some extent, it documents the structure of a system.* |
| `byName`      | Autowiring by property name. *Spring looks for a bean with the same name as the property that needs to be autowired.* For example, if a bean definition is set to autowire by name and it contains a `master` property (that is, it has a `setMaster(..)` method), Spring looks for a bean definition named `master` and uses it to set the property.如果没有匹配的依赖，则不注入也不抛出异常。 |
| `byType`      | Lets a property be autowired if *exactly one bean of the property type* exists in the container. *If more than one exists, a fatal exception is thrown*, which indicates that you may not use `byType `autowiring for that bean. If there are no matching beans, nothing happens (the property is not set). |
| `constructor` | Analogous to `byType` but applies to constructor arguments. If there is not exactly one bean of the constructor argument type in the container, a fatal error is raised。必须要有一个匹配的，或者多个排除一些能够匹配成功的（参考4.5.2小节） |

在byType或constructor自动装配模式中，如果注入数组或集合，所有的的匹配项都会被注入，但是需要的依赖是其他的类型，只能恰好有一个匹配。

*You can autowire strongly-typed* `*Map*`*instances if the expected key type is* `*String*`. An autowired `Map` instance’s values consist of all bean instances that match the expected type, and the `Map` instance’s keys contain the corresponding bean names.（不太懂，不翻译了）

因为byName和byType模式下，如果没有匹配成功的，又因为byName和byType对属性有效（setter方法注入），因此不会抛出异常，也没有成功注入，因此自动装配过后最好进行依赖核查，看有没有注入成功。注意哦，使用<property/>元素指定时没有找到匹配会报错的。

### 自动装配的限制和缺点

- 显示使用<property/>和<constructor-arg/>设置总是会覆盖自动装配的结果
- 自动装配不太精确
- 产生文档的工具不能导出装配信息。
- 如果依赖是数组或集合，多个Bean可能会被自动装配。其他依赖，必须要恰好一个依赖才能装配。

一些解决办法：

- 避免使用自动装配
- 通过autowire-candidate减少可以参与匹配的Bean个数，参考4.5.2小节
- 设置一个<bean/>的属性primary为true，那么多个匹配的Bean中该bean会被注入（优先权高）。
-  通过使用注解得到更加细粒度的控制。

### 从自动装配中排除Bean

<bean/>元素的属性autowire-candidate有三种值：default、false、true。当设置bean的autowire-candidate为false时，该bean不会作为其他Bean依赖的匹配项的考虑范围（也不作为注解@Autowired的考虑范围）。autowire-candidate只影响**基于类型**的自动装配，不影响**基于名字**的自动装配。

也可以设置一群通过了模式匹配的Bean才能作为byType的考虑范围。通过<beans/>元素得default-autowire-candidates属性设置。比如限制可匹配的bean必须以Repository结尾，那么设置属性值为*Repository。如果要使用多个模式，那么以逗号分隔。通过显示设置bean的*autowire-candidate*属性可以覆盖<beans/>的设置。

即使一个Bean设置了*autowire-candidate*为false，但是他本身可以使用自动装配。

## 方法注入

方法注入可以解决Bean之间有依赖关系但生命周期不一致的问题。比如Bean A作用域为singleton，而Bean B作用域为prototype，A的一个方法每次调用时都需要一个Bean B的对象。而容器只会创建一次Bean A，因此Bean A只有一次注入依赖的机会，容器不会为Bean A在每次使用Bean B时都创建一个新的Bean B实例。

[一个牺牲一定IOC的解决方案](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-method-injection)：可以通过实现`ApplicationContextAware` 接口获得容器实例，再通过容器的getBean方法得到新的Bean B实例。

### 查询方法注入

查询方法注入就是通过CGLIB动态产生类的子类，子类覆盖（或实现）要注入的方法，返回查询到的容器中其他Bean的结果（在上面的例子中，就是返回Bean B的新实例）。

由于是继承实现的，因此类和方法不能为final。如果Bean A是通过包扫描得到的，那么Bean A的类的方法必须提供实现，因为抽象类不被扫描到容器中。

要被注入的方法的原型如下，注意，可以不为抽象方法：

```java
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```



举个例子，如下面的类：

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```



然后注入方法：

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```



name指定方法的名字，bean指定方法要返回的Bean，要和方法返回值一致。这样，每次类的process需要用到bean commandProcessor时都会产生新的实例，而不是采用被spring框架侵入的方式。

也可以使用注解@Lookup注入方法，通过指定名字找到Bean：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```



如果@Lookup不提供名字，则默认通过方法返回类型查看Bean。如果使用包扫描则一定要给出方法实现！！！空实现也行。上面没有给出实现，是因为类CommandManager的定义还是通过xml文件给出的。

### 任意方法替代

就是能够任意替换Bean中的方法，听起来好像spring无所不能，，，，但是他就是有这功能，但不常用，这里只给出链接：https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-arbitrary-method-replacement

# [五 Bean作用域](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes)

scope到底是什么，我也不清楚，感觉像生存范围吧。一个有6种scope，其中4种只能在WebApplicationContext中使用。

| Scope                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [singleton](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton) | (*Default*) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `*ServletContext*`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

貌似request、session、application会存在对应的类中，比如ServletRequest、HttpSession、ServletContext中。

## Singleton和Prototype作用域

singleton范围的Bean，在一个容器中只会产生一个实例。是Bean的默认作用域。

prototype作用域的Bean在来一个请求就会创建一个Bean实例。于其他作用域的Bean相比，spring不会管理完整的prototype Bean的生命周期。容器在实例化、配置、装配后交给了client，之后没有了对prototype Bean的记录。因此初始化的生命周期方法都会被调用，但是销毁的生命周期方法不会被调用。在某种程度来说，prototype的角色就像java中new操作符似的。

当用一个singleton Bean，注入一个prototype的依赖时，由于singleton的bean只会进行一次实例化和依赖注入，而如果每次需要一个新的prototype的Bean，依赖注入则不能完成此任务。因此需要使用方法注入，如4.6小节所示。

## request、session、application作用域

和web相关的作用域，需要使用WebApplicationContext容器。为了支持这些作用域，定义Bean之前小小的配置是需要的。在SpringMVC中，你可以web应用中的web.xml文件中声明`DispatcherServlet`, `RequestContextListener`, 或 `RequestContextFilter其中的任意一个都行，这几个。。东西都做了同一件事，就是绑定http请求对象到处理该请求的线程中（我懵懂中。。）。`

request作用域中的Bean每来一个请求时会被创建，不同请求中Bean互不相扰，请求被处理结束后，bean会被销毁。可以通过如下方式配置：

```
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```



或

```
@RequestScope
@Component
public class LoginAction {
    // ...
}
```



session作用域的Bean在会话开启时被创建，不同会话之间互不相扰，会话结束后Bean销毁。通过如下配置：

```
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```



或

```
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```



一个application作用域的Bean在一个web应用中只有一个实例，被存入ServletContext中作为属性。如下配置：

```
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```



或

```
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```



## Scoped Beans作为依赖

短生命周期的作为长生命周期的Bean的依赖，总会出现点问题的。除了使用方法注入，还可以使用AOP代理。通过AOP代理，同样会产生代理类，然后需要注入的Bean都是对代理类操作，代理类会在相应的作用域中取出真正的Bean实例，然后将方法传给真实对象。我猜想不同作用域中的Bean的生命周期都是由容器管理的，因此代理类每次取出都是正确作用域的Bean。

如下面所示，直接在bean中加入<aop:scoped-proxy/>就行了

```
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```



注意，AOP代理默认使用的CGIB，CGIB代理只会拦截public的方法。

也可以设置*proxy-target-class="false"*来使用标准JDK的代理，但是jdk的代理需要提供接口，如下面所示：

```
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```



我猜想。。jdk代理会实现userPreferences接口，而他的实现类则作为调用处理器。。。

# 六 Bean生命周期

Bean不同的生命周期中会执行不同的方法:

![img](.Spring%20IOC/20190223193524552.jpg)

>图片来源：[Spring Bean Life Cycle Tutorial](https://www.concretepage.com/spring/spring-bean-life-cycle-tutorial)
>
>注意点
>
>1. Bean不一定含有上述全部的回调函数
>
>2. 一个`BeanPostProcessor`的方法将作用于所有的Bean, 并且还有一些`aware`方法.
>
>3. `init-method和destroy-method`只能通过xml注解指定. 
>
>   > 如通过`<beans/>`的`default-init-method`和`default-destroy-method`属性可设置该元素下所有Bean的默认初始和销毁函数，可被`<bean/>`元素的`init-method`和`destroy-method`属性覆盖
>
>4. 注解方式只能使用`@PostConstruct`和`@PreDestroy`了

下面三种生命周期常用于初始化

- The [`InitializingBean`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) and [`DisposableBean`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) callback interfaces
- Custom `init()` and `destroy()` methods
- The [`@PostConstruct` and `@PreDestroy` annotations](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations). You can combine these mechanisms to control a given bean.

> 如果不同生命周期方法调用不同的函数，那么会以一定顺序执行。如果不同生命周期方法调用相同函数，比如`init()`，那么该方法只会被调用一次。

在WebApplicationContext应用中，已经能够优雅的关闭ioc，而非web应用需要自己注册个关闭钩子（shutdown hook），否则有些Bean的生命周期不被调用：

```java
public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

还有什么Aware接口，继承后可以通过接口方法获得一些对象，比如容器本身啥的。

> 参考[Bean生命周期](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-nature)

# [七 Bean定义继承](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions)

Bean的定义是可以继承和覆盖的，但有些属性不会被继承，有点小复杂因此不建议使用。。

```
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如上面所示，子Bean通过parent属性指定父bean，父bean貌似可以不设置abstract，但是没有指定class属性时必须设置abstract，如果不设置那么父bean也会被初始化。

# [八 容器扩展](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-extension)

可以通过实现ApplicationContext来扩展容器功能，但不建议，一般都是继承相应的接口来扩展spring容器功能。

BeanPostProcessor可以自定义Beans，它会在所有的bean初始化前和初始化后分别调用它自己的逻辑，逻辑中会处理实例化的Bean。比如spring的自动代理就是通过该接口扩展容器功能的，容器的注解配置功能也是该接口实现类扩展的（扫描Bean里面的注解，然后实现一定逻辑）。需要实现BeanPostProcessor，然后和普通Bean一样在元数据配置中配置，容器会先实例化该接口实现类。注意，BeanPostProcessor作用域在容器中，只能处理一个容器中的所有Bean（除了BeanPostProcessor和接下来讲的处理器）。

BeanFactoryPostProcessor操作元数据配置，就是说BeanFactoryPostProcessor先读取元数据，然后可以对元数据进行修改。其他的和BeanPostProcessor一样，在元数据中配置该接口实现类呀，容器范围啦。

FactoryBean（不要和BeanFactory混淆）可以控制容器的实例化逻辑，实现该接口的类会被spring当做工厂bean，可以生产其他bean。当对象构造十分复杂，适合使用java代码而不是xml配置时，可以采用该方法定义工厂，然后生产该对象。请查阅本博文3.2.4小结对FactoryBean使用的介绍。

# 九 基于注解的容器配置

下面讲的主要是关于依赖注入的注解，和bean定义有关的注解在后面会提到。如果要在xml配置中使用注解的功能，需要开启注解的处理器

```xml
<context:annotation-config/>
```

该注解默认开启bean post-processor，包括 AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor, PersistenceAnnotationBeanPostProcessor, and RequiredAnnotationBeanPostProcessor。关于BeanPostProcessor，第八小节有提到过。注意！！！`<context:annotation-config/>`只会查看同**一个容器**中的bean的注解。

> 参考[基于注解的容器配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)

## @Required

@Required注解到setter方法上，表示该属性必须要被注入依赖。

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

## @Autowired

`@Autowired`可以注解到属性、任意参数方法、构成函数上，通过类型注入，拥有着比XML配置更加细腻度的控制。并且默认将被`@Autowire`注解的字段、方法、构造函数视为必须的依赖！如果没有匹配成功则抛出异常。

注释构造函数，任意参数都行：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

注释setter方法：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

注释方法，任意参数都行：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

注释字段，尽管是**private**！！！同时还可以**和构造函数注入混合使用**：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> 但是, 构造函数执行结束后才注入字段. 因此, 在构造函数中拿不到字段值.

如果是数组，那么所有类型匹配成功的Bean都会被注入，集合也是一样：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

}
```



map实例也可以自动注入，需要key类型为String，那么所有匹配的Bean会作为map的value，bean标识符作为key值：

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认被@Autowired注解的方法、构造函数、字段是必须的，可以设置为不必须的：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

## 通过@Primary微调自动注入

@Autowired通过类型自动注入依赖，但是类型匹配成功的Bean太多，可以使用@Primary注解指定一个Bean具有优先权，该Bean会被注入。

配置通过java代码生成的Bean：

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```



配置xml中的Bean：

```
    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>
```



配置包扫描到的Bean：

```
 @Primary
 @Component
 public class A{


 }
```



## 通过@Qualifier微调@Autowired

### 介绍

`@Autowired`注入, 且多个Bean类型与之匹配时, `@Qualifier`将再次根据**限定名**筛选符合匹配的Bean.

* 因为Bean的限定名可以重复, 因此`@Qualifier`只是缩小了匹配的Bean的范围
* 如果Bean没有定义限定名，那么他的Bean标识符会被当做默认限定名。

> 如果向指定一个唯一的Bean标识符作为依赖注入，应该考虑`@Resource`，而不是`@Autowired`。

### 使用

* 微调
  * 注解到字段上
  	```java
    public class MovieRecommender {

        @Autowired
        @Qualifier("main")
        private MovieCatalog movieCatalog;

        // ...
    }
    ```
  * 注解到参数上
  	```java
    public class MovieRecommender {

        private MovieCatalog movieCatalog;

        private CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
                CustomerPreferenceDao customerPreferenceDao) {
            this.movieCatalog = movieCatalog;
            this.customerPreferenceDao = customerPreferenceDao;
        }

        // ...
    }
    ```
* 配置限定名
  * XML
  	```xml
    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>
    ```
  
  * Java: 这个不知道

## @Resource

@Resource通过名字注入，只能注解在字段和setter方法上。默认使用@Resource时没有给出Bean名，则会使用属性名

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```



默认使用属性名movieFinder：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```



# [十 classpath扫描和component管理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-classpath-scanning)

第九节讲的是和自动注入相关的注解，尽管这也是元数据配置的一部分，但是没有介绍如何使用注解得到Bean定义。接下来便会将到。

stereotype注解包含: `@Component`, `@Service`、`@Repository`, and `@Controller`。被注解的类会被扫面成Bean定义。`@Component`是最泛化的概念，而`@Repository`, `@Service`, and `@Controller`拥有比较具体的语义，分别用于持久层、服务层和控制层。使用这些注解时可以提供Bean的名字，如果没有提供，会将类名的第一个字母转为小写后作为Bean名字。通过@Scope可以设置作用域。通过@Qualifier可以提供限定名。

spring提供的很多注解都是元注解，因为可以组合多个注解构造新的注解。比如`@Service`定义中被`@Component`注解过，因此包扫描时会将`@Service`注解的类扫描成Bean定义。又如`@RestController`是`@Controller` 和 `@ResponseBody`的组合。组合注解也可以自定义元注解，暴露元注解的部分属性，比如`SessionScop`e，可查阅相关内容。

通过包扫面可以找到`@Component`注解的Bean定义，如果使用基于java代码的方式提供元数据，可以通过如下方法开启包扫描：

```java
@Configuration
@ComponentScan("org.example")
public class AppConfig  {
    ...
}
```

如果是XML方式：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

> `<context:component-scan>`隐式使能`<context:annotation-config>`的功能。

可以通过Component提供Bean定义，也就是在@Component注解的类中通过@Bean提供Bean元数据。@Bean注解的方法类似于3.2小节中的方法工厂，也可以注解静态方法，则为静态工厂，静态工厂不会造成类的实例化。@Bean默认使用类型匹配的注入方法，如果有多个匹配项，可以通@Qualifier和@Primary解决。对方法的注入相当于对构造函数的注入一样。在@component中使用@Bean不会进行代理处理，因此最好不要在工厂方法中调用另一个工厂方法，参考[这篇博客](https://blog.csdn.net/jdbdh/article/details/82795049)。下面是一个简单例子：

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

# [十一 基于java的容器配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java)

在基于java的容器配置中需要使用到@Configuration和@Bean。@Configuration表明这是一个配置类，里面被@Bean注释的方法会产生Bean实例。尽管@Bean可以在@Component中使用，但是最主要的还是配合@Configuration使用。@Configuration被@Component注解过，因此也可被当做一个Bean定义。在@Configuration注解的类会被CGIB动态继承该类，重写@Bean注解的方法，该方法每次只返回该作用域内的Bean，维护了Bean的作用域，而在@Component中就是个普通方法。因此使用@Configuration注解，可以在一个@Bean方法内调用其他@Bean方法。

## @Configuration和容器实例化

* 例子

    ```java
    @Configuration
    public class AppConfig {

        @Bean
        public MyService myService() {
            return new MyServiceImpl();
        }
    }
    ```
    
* 包扫描

    ```java
    @Configuration
    @ComponentScan(basePackages = "com.acme") 
    public class AppConfig  {
        ...
    }
    ```

* 貌似, 配置类中的静态方法也会被扫描与注册

* 实例化
	
	通过`AnnotationConfigApplicationContext`建立容器，传入配置类：
  
  ```java
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        MyService myService = ctx.getBean(MyService.class);
        myService.doStuff();
    }
	```
  
	`AnnotationConfigApplicationContext` 不局限于`@Configuration`类，还可以是`@Componet`：
  
    ```java
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
        MyService myService = ctx.getBean(MyService.class);
        myService.doStuff();
    }
    ```

> 一般web应用是通过xml文件配置的，也可通过Java代码配置，参考：[Support for Web Applications with AnnotationConfigWebApplicationContext](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-instantiating-container-web)

## @Bean

* Bean注册

  `@Bean`注解的方法的返回值为Bean的类型，Bean名字和方法名一致。

* 依赖注入

  * 注入方式基本与构造函数注入一致, 同时无需`@Autowire`注解

  * 貌似, 注入依赖时, 会先根据类型注入, 接着按照Bean名字.

> 参考[@Bean](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-bean-annotation)

## @Import

@Import可以导入其他配置类的Bean定义：

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```



## java和xml混合配置

@Configuraion类并不会100%替代xml配置，相反，他们可以相互配合使用。

### xml配置中使用@Configuration类

首先有一个configuration配置类：

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

可以将该类作为Bean在xml声明，需要通过`<context:annotation-config/>`开启注释，容器会识别注解，执行注解的逻辑。

```xml
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>

    <bean class="com.acme.AppConfig"/>

</beans>
```

因为`@Configuration`被`@Conmponent`注释过的，因此可以开启包扫描扫描该配置类：

```xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>

</beans>
```

### @Configuration类中使用xml

通过@ImportResource可以导入xml文件定义的Bean

```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

}
```

# 十二 其他

## 父子容器

一个容器可以有父容器，比如springMVC中就有两个容器，一个web应用对应一个ApplicationContext容器，为父容器，每个DispatcherServlet对应一个WebApplicationContext容器，为子容器。父容器中定义的Bean都可以被子容器访问到，也可被子容器隐藏。在ApplicationContext中可以定义和业务逻辑、数据访问有关的Bean，而WebApplicationContext 中可以定义和控制器与视图解析器有关的Bean.

> 参考
>
> * [What is the difference between spring parent context and child context?](https://stackoverflow.com/questions/43452644/what-is-the-difference-between-spring-parent-context-and-child-context)
>* [About multiple containers in spring framework](https://stackoverflow.com/questions/18578143/about-multiple-containers-in-spring-framework)

## 好像构造函数注入了, 字段就不再注入



# 参考

* [官方文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans)