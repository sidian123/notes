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
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

## 配置

* 启动Spring Session

    ```java
    @EnableRedisHttpSession 
    public class Config {
    }
    ```

	>  `@EnableRedisHttpSession`注解启用了Spring Session, 并使用Redis做存储, 

* Spring Session配置

    > 默认就好
    
    ```properties
    spring.session.store-type=redis # Session store type.
    server.servlet.session.timeout= # Session timeout.
    spring.session.redis.flush-mode=on-save # Sessions flush mode.
    spring.session.redis.namespace=spring:session # Namespace for keys used to store sessions.
    ```
    
* Redis配置

    ```properties
    spring.redis.host=localhost  #Server host
    spring.redis.password=    #password
    spring.redis.port=6379    #Redis server port
    ```

# 使用

* 方式一(传统方式)

  通过Spring MVC直接注入





