# 介绍

Swagger是一种定义接口相关信息的**规范**, 通过yml或json格式的文件(**描述文件**)来描述这种信息.

Springfox是Swagger规范的一种实现, 提供了自动扫描代码并生成描述文件, 也提供了将描述文件转化为网页UI的功能.

开发人员可以在Springfox提供的网页上测试接口, 获取接口详细信息.

# Maven依赖

Spring boot中引入依赖

```xml
<!- 提供扫描代码, 得到Swagger描述内容的功能 ->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!- 提供将描述内容转化为网页的功能 ->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

# 配置

```java
@Configuration
@EnableSwagger2
public class SpringFoxConfig {                                    
    @Bean
    public Docket api() { 
        return new Docket(DocumentationType.SWAGGER_2)//使用Swagger2规范
          .select()
          .apis(RequestHandlerSelectors.any())//扫描所有请求处理器
          .paths(PathSelectors.any())//扫描所有路径
          .build();             
    }
}
```

# 使用

* 访问描述内容

  地址`/v2/api-docs`, 如

  ```url
  http://localhost:8080/v2/api-docs
  ```

* 访问网页UI

  地址`/swagger-ui.html`, 如

  ```url
  http://localhost:8080/swagger-ui.html
  ```

# 注解

不用注解, Swagger也会解析出信息. 但注解能够提供更多的信息来.

略



# 参考

* [Swagger介绍及使用](https://www.jianshu.com/p/349e130e40d5) 侧重Swagger缘由脉络
* [Setting Up Swagger 2 with a Spring REST API](https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api) 侧重Swagger使用
* [Springboot集成Swagger操作步骤](https://www.jianshu.com/p/be1e772b089a)





