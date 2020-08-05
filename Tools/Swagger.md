# 介绍

Swagger是一种定义接口相关信息的**规范**, 通过yml或json格式的文件(**描述文件**)来描述这种信息.

Springfox是Swagger规范的一种实现, 提供了自动扫描代码并生成描述文件, 也提供了将描述文件转化为网页UI的功能.

开发人员可以在Springfox提供的网页上测试接口, 获取接口详细信息.

# Maven依赖

Spring boot中引入依赖

```xml
<!-- 提供扫描代码, 得到Swagger描述内容的功能 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- 提供将描述内容转化为网页的功能 -->
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

> 注意, 并不是所有Swagger注解都使用, 也不是所有的注解属性都使用. 

下面将介绍常用注解的使用.

> 注意, 注解仅作描述, 无额外功能.

## Controller相关

* `@Api`

  描述类

  * `tags` 类名
  * `description` 类详细信息

  > 常用`description`

  ```java
  @Api(tags="测试类")
  @RestController
  public class TestController {
  	//...
  }
  ```

* `@ApiOperation`

  描述方法

  * `value` 方法总结, 默认方法名.
  * `notes` 方法详细说明
  * `httpMethod` 请求方式, Swagger会从Spring注解上获取信息.

  ```java
  @ApiOperation(value="say hello",notes="say hello to the man")
  @GetMapping("/hello")
  public String hello(String man){
      return "hi, "+man;
  }
  ```

* `@ApiParam`

  描述参数

  * `value` 参数说明 
  * `required` 是否必填, 默认false

  ```java
  @ApiOperation(value="say hello",notes="say hello to the man")
  @GetMapping("/hello")
  public String hello(
      @ApiParam(value= "这个人的名字",required = true) 
      @RequestParam(required = true) 
      String man
  ){
      return "hi, "+man;
  }
  ```

  > `@ApiParam`仅含语义, 并无约束, 所以常常配置`@RequestParam`注解使用, 达到描述与其功能一致性
  
  请最好配合`@RequestParam`使用, 否则参数将被当作请求体参数!!!
  
* 另一种请求参数描述方式

  ```java
  @ApiOperation(value = "分页获取数据列表", notes = "分页获取数据列表", produces = "application/json")
  @ApiImplicitParams({
      @ApiImplicitParam(name = "tid", value = "Token Id", dataType = "String", required = true, paramType = "header"),
      @ApiImplicitParam(name = "name", paramType = "name", dataType = "String"),
      @ApiImplicitParam(name = "type", paramType = "数据类型,1-指南,2-普通文本", dataType = "int"),
      @ApiImplicitParam(name = "pageNum", paramType = "query", dataType = "int", defaultValue = "1"),
      @ApiImplicitParam(name = "pageSize", paramType = "query", dataType = "int", defaultValue = "10"),
  })
  @RequestMapping(value = "/data", method = RequestMethod.GET)
  public HttpResponseTemp<ItemsPageEntity<DataResultEntity>> getDataListByPage(
      @RequestParam(value = "name", required = false) String name,
      @RequestParam(value = "type", required = false) Integer type,
      @RequestParam(value = "pageNum", required = false, defaultValue = "1") int pageNum,
      @RequestParam(value = "pageSize", required = false, defaultValue = "10") int pageSize) {
      ControllerHelper.validatePageSize(pageNum, pageSize);
      ItemsPageEntity<DataResultEntity> itemsPageEntity = dataService.getDataList(name, type, pageNum, pageSize);
      return HttpResponseTemp.success(itemsPageEntity);
  }
  ```

  

## Model相关

实体类被用作请求参数或响应时, 注解信息将被Swagger提取

* `@ApiModel`

  描述实体类

  * `value` 实体名
  * `description` 详细描述

* `@ApiModelProperty`

  实体字段描述

  * `value` 字段说明
  * `required` 是否必填
  * `example` 为该字段的值举个例子
  * `hidden` 不显示该字段

# 其他

## knife4j

[knife4j](https://gitee.com/xiaoym/knife4j) 也是一种Swagger文档的呈现形式, 界面内容更充实, 美观. 推荐使用

文档地址: `/doc.html`

* `pom.xml`

  ```xml
  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.9.2</version>
  </dependency>
  <dependency>
      <groupId>com.github.xiaoymin</groupId>
      <artifactId>knife4j-spring-boot-starter</artifactId>
      <version>2.0.3</version>
  </dependency>
  ```

* 配置

  ```properties
  ## 开启Swagger的Basic认证功能,默认是false
  knife4j.basic.enable=true
  ## Basic认证用户名
  knife4j.basic.username=admin
  ## Basic认证密码
  knife4j.basic.password=123456
  
  #swagger page url
  swagger.doc-url=http://localhost:8013/
  #是否激活 swagger true or false
  swagger.is.enable=true
  ```

* 使用示例

  ```java
  @Configuration
  @EnableKnife4j
  @EnableSwagger2
  public class SwaggerConfiguration {
      @Value("${swagger.doc-url:http://localhost:8200}")
      private String swaggerPageUrl;
      @Value("${swagger.is.enable:true}")
      private boolean swaggerIsEnable;
  
      @Bean(value = "1.知识构建服务")
      public Docket kaAdminApi() {
          return new Docket(DocumentationType.SWAGGER_2)
                  .enable(swaggerIsEnable)
                  .apiInfo(apiInfo())
                  .groupName("1.知识构建服务")
                  .select()
                  .apis(RequestHandlerSelectors.basePackage("com.ka.rest"))
                  .paths(PathSelectors.any())
                  .build();
      }
  
      @Bean(value = "2.Admin API")
      public Docket adminDocket() {
  
          return new Docket(DocumentationType.SWAGGER_2)
                  .enable(swaggerIsEnable)
                  .apiInfo(apiInfo())
                  .groupName("2.admin api")
                  .select()
                  .apis(RequestHandlerSelectors.basePackage("com.core.rest"))
                  .paths(PathSelectors.any())
                  .build();
  
      }
  
      @Bean(value = "3.schema展示服务")
      public Docket schemaApi() {
          return new Docket(DocumentationType.SWAGGER_2)
                  .enable(swaggerIsEnable)
                  .apiInfo(apiInfo())
                  .groupName("3.schema展示服务")
                  .select()
                  .apis(RequestHandlerSelectors.basePackage("com.schema.rest"))
                  .paths(PathSelectors.any())
                  .build();
      }
  
      private ApiInfo apiInfo() {
  
          Contact contact = new Contact("医疗信息技术(杭州)有限公司", "", "1234@qq.com");
  
          return new ApiInfoBuilder()
                  .title("服务服务接口文档")
                  .description("服务服务接口文档")
                  .termsOfServiceUrl(swaggerPageUrl)
                  .contact(contact)
                  .version("1.0")
                  .build();
  
      }
  }
  ```

# 参考

* [Swagger介绍及使用](https://www.jianshu.com/p/349e130e40d5) 侧重Swagger缘由脉络
* [Setting Up Swagger 2 with a Spring REST API](https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api) 侧重Swagger使用

* [swagger2常用注解说明](https://www.cnblogs.com/yueguanguanyun/p/9041690.html) Swagger注解详细使用介绍. ( 排版有点乱 )
* [Swagger-Core Annotations](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X) 官方对常用注解的介绍, 很多内容国内开发用不到.

* [Springboot集成Swagger操作步骤](https://www.jianshu.com/p/be1e772b089a)





