# 一 介绍
> 预备知识：
>
> * HttpEntity含有headers和body信息；子类ResponseEntity添加了状态码；子类RequestEntit添加了method和url信息。
>
> * 都有对象的Builder类，方便构建对象。

* 在一些HTTP客户端库的基础上提供更高层的API，使得更容易的访问rest风格的http请求。

* http请求客户端、线程安全、同步请求

* 底层基于JDK HttpURLConnection（默认）或Apache HttpComponents等等

* 依赖引入

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```

  Spring Boot >= 1.4 后, 需手动注入`RestTemplate`, 如

  ```java
  @Bean
  public RestTemplate restTemplate(RestTemplateBuilder builder) {
     // Do any additional configuration here
     return builder.build();
  }
  ```

# 二 正文

* **方法**：

  * **通用方法**：`exchange`和`execute`，用于支持更少见的请求，可定制的功能更多（可定制化程度：`execute`>`exchange`），一个比较底层的方法。

  * **Rest方法**：该类的方法被组织成rest方式的风格，命名方式有规律，部分方法如下所示：
    
    | HTTP    | RestTemplate                                 |
    | :------ | :------------------------------------------- |
    | DELETE  | `delete(String, String...)`                  |
    | GET     | `getForObject(String, Class, String...)`     |
    | HEAD    | `headForHeaders(String, String...)`          |
    | OPTIONS | `optionsForAllow(String, String...)`         |
    | POST    | `postForLocation(String, Object, String...)` |
    | PUT     | `put(String, Object, String...)`             |
    
    * 基本上，方法名第一部分表示http请求方法，第二部分表示返回值。
    * 基本上，第一个参数为url，接着（请求对象）request（if any），接着返回对象类型（if any），接着第三个为url参数（if any）。
      
      * request参数一般为`HttpEntity`，或者`MultiValueMap`（可写入`HttpEntity`中），~~但不可为其他类型对象~~ 也可为其他对象, 只要存在对应转化器(Converter)
      
      * 当不想要返回值时，返回类型可设为`Void.class`
      
    * 这些方法底层都是调用了`doExecute`方法实现的

* **初始化**：底层默认使用JDK的`HttpURLConnection`实现，也可在构造函数上切换其他HTTP库。内置支持：

  * Apache HttpComponents
  * Netty
  * OkHttp

  使用Apache的例子：

  ```java
  RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
  ```

* **URI**：方法中第一个参数指定URI，它是一个模板，**支持路径变量**，方法的最后一个参数设置变量值，并且**会自动进行url编码**。一个例子如下所示：

  ```java
  String result = restTemplate.getForObject("https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");
  ```

* **Headers**：`exchange`方法可以设置请求头部，也会获取响应头部。为啥rest方法不用设置呢？因为能自动推断，所以通常情况下不用设置，见Body部分。

* **Body**（消息体）：请求消息体中填充对象或响应消息体中获取对象，都是通过`HttpMessageConverter`完成的。通常情况下，你是不必显示设置`Content-Type`和`Accept`，因为仅通过传入的Java类型就可以确定转换器，然后由转换器自己设置这些头字段。

  * 方法参数的请求对象`Object`的类型，可以确定序列化请求对象的转换器，继而转换器确定`Content-Type`
  * 方法参数的结果对象类型`responseType`可以确定解析响应消息体的转换器，继而转换器确定`Accept`
  * 当然可以通过`exchange`手动指定，这会影响使用的转换器的选择。

* [HttpMessageConverter](<https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html>)：Spring为常用的MIME类型和Java类型注册了一些默认的转换器，至于该转化器是否生效，取决于classpath下是否有对应的jar包。默认转化器如下所示：
  
  | MessageConverter                         | Description                                                  |
  | :--------------------------------------- | :----------------------------------------------------------- |
  | `StringHttpMessageConverter`             | An `HttpMessageConverter` implementation that can read and write `String`instances from the HTTP request and response. By default, this converter supports all text media types (`text/*`) and writes with a `Content-Type` of `text/plain`. |
  | `FormHttpMessageConverter`               | An `HttpMessageConverter` implementation that can read and write form data from the HTTP request and response. By default, this converter reads and writes the `application/x-www-form-urlencoded` media type. Form data is read from and written into a `MultiValueMap<String, String>`. |
  | `ByteArrayHttpMessageConverter`          | An `HttpMessageConverter` implementation that can read and write byte arrays from the HTTP request and response. By default, this converter supports all media types (`*/*`) and writes with a `Content-Type` of `application/octet-stream`. You can override this by setting the `supportedMediaTypes` property and overriding `getContentType(byte[])`. |
  | `MarshallingHttpMessageConverter`        | An `HttpMessageConverter` implementation that can read and write XML by using Spring’s `Marshaller` and `Unmarshaller` abstractions from the `org.springframework.oxm` package. This converter requires a `Marshaller` and `Unmarshaller` before it can be used. You can inject these through constructor or bean properties. By default, this converter supports `text/xml` and `application/xml`. |
  | `MappingJackson2HttpMessageConverter`    | An `HttpMessageConverter` implementation that can read and write JSON by using Jackson’s `ObjectMapper`. You can customize JSON mapping as needed through the use of Jackson’s provided annotations. When you need further control (for cases where custom JSON serializers/deserializers need to be provided for specific types), you can inject a custom `ObjectMapper` through the `ObjectMapper` property. By default, this converter supports `application/json`. |
  | `MappingJackson2XmlHttpMessageConverter` | An `HttpMessageConverter` implementation that can read and write XML by using[Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml) extension’s `XmlMapper`. You can customize XML mapping as needed through the use of JAXB or Jackson’s provided annotations. When you need further control (for cases where custom XML serializers/deserializers need to be provided for specific types), you can inject a custom `XmlMapper` through the `ObjectMapper` property. By default, this converter supports `application/xml`. |
  | `SourceHttpMessageConverter`             | An `HttpMessageConverter` implementation that can read and write`javax.xml.transform.Source` from the HTTP request and response. Only `DOMSource`, `SAXSource`, and `StreamSource` are supported. By default, this converter supports `text/xml` and `application/xml`. |
  | `BufferedImageHttpMessageConverter`      | An `HttpMessageConverter` implementation that can read and write`java.awt.image.BufferedImage` from the HTTP request and response. This converter reads and writes the media type supported by the Java I/O API. |
  
  从描述中，可以看到转换器会自动设置头字段。其中注意的是，请求体中直接转入pojo对象，会被jackson转化为JSON数据，如果想使用`FormHttpMessageConverter`生成前端表单post请求格式的数据，需要使用`MultiValueMap`类存储数据。
  
  > spring mvc（服务端）和RestTemplate（客户端）中都用到了`HttpMessageConverter`、`RequestEntity`/
  
* **form表单**：表单的消息体由`FormHttpMessageConverter`转换器进行转换，当请求数据为[MultiValueMap<String,?>](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/util/MultiValueMap.html)类时会使用该转换器，该类可以使用[MultipartBodyBuilder](<https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/client/MultipartBodyBuilder.html>)更方便的创建出来。表单有两种编码方式：

  * `application/x-www-form-urlencoded`：当`MultiValueMap`的**值**（value，非key、非请求参数值）全为字符串时，转换器会使用这种编码方式。

  * `multipart/form-data`：其他情况都使用这种编码方式。每个part都是一个HttpEntity，可能会有头字段，如文件须有`Content-Type`，但一般不必手动设置，每个part的转换器会自动设置，如下面的例子所示：

    ```java
    MultipartBodyBuilder builder = new MultipartBodyBuilder();
    builder.part("fieldPart", "fieldValue");
    builder.part("filePart", new FileSystemResource("...logo.png"));
    builder.part("jsonPart", new Person("Jason"));
    
    template.postForObject("https://example.com/upload", builder.build(), Void.class);
    ```

    上面的序列化文件时，使用的，，是哪个转换器？？不知道，，反正使用的对象必须有对应的转换器存在才行！

# 三 例子

测试时所用的，全贴下来了，，

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class BlogApplicationTests {

    

    static class A{
        int a;
        int b;

        public int getA() {
            return a;
        }

        public void setA(int a) {
            this.a = a;
        }

        public int getB() {
            return b;
        }

        public void setB(int b) {
            this.b = b;
        }
    }



    @Test
    public void test3(){
        A a=new A();
        a.setA(1);
        a.setB(2);
        new RestTemplate().postForObject("http://localhost:8080/image/test",a,Void.class);
    }
    @Test
    public void test4(){
        /*MultiValueMap<String,Object> map=new LinkedMultiValueMap<>();
        map.add("a","a");
        map.add("b","23");*/
        MultipartBodyBuilder builder=new MultipartBodyBuilder();
        builder.part("a","a");
        builder.part("b","b");
        builder.part("file",new FileSystemResource("C:\\Users\\Administrator\\Pictures\\王通.jpg"));

        new RestTemplate().postForObject("http://localhost:8080/image/test3",builder.build(),Void.class);

    }
    @Test
    public void test5(){
        Map<String,Object> map=new HashMap<>();
        map.put("a","aa");
        map.put("b",23);

        new RestTemplate().postForObject("http://localhost:8080/image/test2",new HttpEntity<Map>(map),Void.class);
    }

    @Test
    public void contextLoads() throws IOException {
        RestTemplate restTemplate = new RestTemplate();
        String url
                = "http://localhost:8080/user/login";
        MultiValueMap<String,String> param= new LinkedMultiValueMap<>();
        param.add("username","千霜");
        param.add("password","123456");

        HttpHeaders headers=new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        HttpEntity<Map> entity=new HttpEntity<>(param,headers);

        ResponseEntity<Status> responseEntity=restTemplate.postForEntity(url,entity,Status.class);
        Status status=responseEntity.getBody();
        System.out.println(status);
    }



    @Test
    public void test2() throws FileNotFoundException, URISyntaxException {
        MultipartBodyBuilder builder=new MultipartBodyBuilder();
        builder.part("file",new FileSystemResource("C:\\Users\\Administrator\\Pictures\\王通.jpg"));

        Status status=new RestTemplate().postForObject("http://localhost:8080/image/upload",builder.build(),Status.class);
        System.out.println(status);
    }
    //new FileInputStream("C:\\Users\\Administrator\\Pictures\\王通.jpg")
}
```

# 五 其他

## bug

使用过程中发现了点spring boot的bug，就是必须添加了如下依赖后，一些功能才能使用：

```xml
<dependency>
	<groupId>org.reactivestreams</groupId>
	<artifactId>reactive-streams</artifactId>
	<version>1.0.2</version>
</dependency>
```

见[issue12579](<https://github.com/spring-projects/spring-boot/issues/12579>)

## 返回`List<T>`

一般对象的序列化都是由Jackson完成的, 当解序列化复杂类型对象时, 需要特定的`ParameterizedTypeReference<T>`记录泛型类型信息. `RestTemplate`也刚好提供了对应方法. 使用例子如下:

```java
List<Student> students= (List<Student>) restTemplate.exchange(URI_PREFIX + "/", HttpMethod.GET, null, new ParameterizedTypeReference<List<Student>>() {}).getBody();
```

如果不想这个复杂, 可以使用数组:

```java
Student[] students = restTemplate.getForObject(URI_PREFIX + "/", Student[].class);
```

> 参考:[ClassCastException: RestTemplate returning List instead of List](https://stackoverflow.com/questions/19463372/classcastexception-resttemplate-returning-listlinkedhashmap-instead-of-listm)

# 参考
* [REST Endpoints office doc](<https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#rest-client-access>)
* [RestTemplate office post](<https://spring.io/blog/2009/03/27/rest-in-spring-3-resttemplate>)
* [RestTemplate Baeldung](<https://www.baeldung.com/rest-template>)
* [MIME](<https://blog.csdn.net/jdbdh/article/details/83932406#44MIME_495>)
* [on Basic Auth with RestTemplate](https://www.baeldung.com/how-to-use-resttemplate-with-basic-authentication-in-spring)：进阶阅读，保存凭证credentials