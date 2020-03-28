# 介绍

Spring替代Servlet容器自身会话的解决方案. 

## 特性

* 替换了`HttpSession`的实现, 可使用Redis, 数据库或Hazelcast作为会话存储. 
* 接收WebSocket消息时,  能够保证会话存货
* 支持Spring WebFlux的会话使用

## 依赖

这是使用Redis作为存储的Maven依赖

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>


<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

> 这里直接使用Spring Data Redis的依赖来引入Redis自动配置

## 配置

* 启动Spring Session

    ```java
    @EnableRedisHttpSession 
    public class Config {
    }
    ```

	>  `@EnableRedisHttpSession`注解启用了Spring Session, 并使用Redis做存储, 

* Spring Session配置

    必须的配置
    
    ```properties
    # Session store type.
    spring.session.store-type=redis
    ```
    
    下面的可选, 默认就好
    
    ```properties
    # Session timeout.
    server.servlet.session.timeout=
    # Sessions flush mode.
    spring.session.redis.flush-mode=on-save
    # Namespace for keys used to store sessions.
    spring.session.redis.namespace=spring:session
    ```
    
* Redis配置

    ```properties
    #Server host
    spring.redis.host=localhost
    #password
    spring.redis.password=
    #Redis server port
    spring.redis.port=6379
    ```

# 使用

* 传统方式

  通过Spring MVC直接注入
  
  ```java
  @RequestMapping("/test")
  public String test(HttpServletRequest request){
      HttpSession session = request.getSession();
      return session.getId();
  }
  ```
  
* 注入

  ```java
  @Autowired
  HttpSession session;
  
  @RequestMapping("/test")
  public String test(HttpServletRequest request){
     return session.getId();
  }
  ```

* 其他方式, 如`SessionRepository`

  不会用...

# 参考

* [Spring Session官网](https://docs.spring.io/spring-session/docs/2.2.2.RELEASE/reference/html5/#introduction)
* [Spring-Session基于Redis管理Session](https://segmentfault.com/a/1190000015432590)





