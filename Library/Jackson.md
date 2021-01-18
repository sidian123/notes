[TOC]

# 一 介绍
jackson用于在java对象与JSON之间映射。jackson有三种处理JSON的处理模型：
* **Data Binding**：在JSON与简单对象（POJO，Maps, Lists, Strings, Numbers, Booleans 和null）或更复杂一点的对象（如对象中还有其他复杂对象）之间转换。
* **Tree Model**：在内存中，JSON被表示为树的形式，便于遍历。
* **Streaming Model**：以流的方式读取字段或值. 前面两种处理模型都是基于该模型的，使用它主要是为了获得更高的性能或更细致的控制。

这里讲述Data Binding模型。

# 二 Maven配置
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.11.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.11.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.2</version>
</dependency>
```
# 三 使用

## Data Binding

主要使用`ObjectMapper`类，通过`readValue`方法解析JSON，可以从网络（`URL`）、文件或字符串上获得JSON，使用时需要传入对应的java类型；通过`writeValue`方法生成JSON，可以写入到字符串、文件中。

JSON类型和java类型的对应关系：

| JSON Type         | Java Type                   |
| ---------------- |  -------------------------- |
| object            | Map或Pojo甚至复杂对象       |
| array             | List                        |
| string            | String                      |
| complete number   | Integer, Long or BigInteger |
| fractional number | Double / BigDecimal         |
| true \| false     | Boolean                     |
| null              | null                        |

对于object类型的JSON，可以使用Map、pojo，或者更复杂的对象。如果复杂对象和Map混用，需要用到泛型，由于编译后类型信息丢失，我们需要`TypeReference`类保存泛型信息。

### POJO与JSON
假设有POJO类：
```java
// Note: can use getters/setters as well; here we just use public fields directly:
public class MyValue {
  public String name;
  public int age;
  // NOTE: if using getters/setters, can keep fields `protected` or `private`
}
```
创建ObjectMapper实例：
```java
ObjectMapper mapper = new ObjectMapper(); // create once, reuse
```
通过实例解析JSON到java对象，JSON来源多样，注意要传入java对象的类型：
```java
MyValue value = mapper.readValue(new File("data.json"), MyValue.class);
// or:
value = mapper.readValue(new URL("http://some.com/api/entry.json"), MyValue.class);
// or:
value = mapper.readValue("{\"name\":\"Bob\", \"age\":13}", MyValue.class);
```
如果想要生成JSON的话：
```java
mapper.writeValue(new File("result.json"), myResultObject);
// or:
byte[] jsonBytes = mapper.writeValueAsBytes(myResultObject);
// or:
String jsonString = mapper.writeValueAsString(myResultObject);
```
### Map,List与JSON
如果JSON类型比较简单，则不用传入java泛型信息：
```java
//如果JSON为数值
Map<String, Integer> scoreByName = mapper.readValue(jsonSource, Map.class);
//如果JSON为简单object
Map<String, Object> scoreByName2 = mapper.readValue(jsonSource, Map.class);
//如果JSON为简单数组
List<String> names = mapper.readValue(jsonSource, List.class);

// and can obviously write out as well
mapper.writeValue(new File("names.json"), names);
```
如果JSON比较复杂，那么Map或List的值类型也比较复杂，需要使用TypeReference传入泛型信息：
```java
Map<String, ResultValue> results = mapper.readValue(jsonSource,
   new TypeReference<Map<String, ResultValue>>() { } );
// why extra work? Java Type Erasure will prevent type detection otherwise
```
## Tree Model

## Streaming Model

> 参考[Reading and Writing Event Streams](http://www.cowtowncoder.com/blog/archives/2009/01/entry_132.html)

### 读取

* 使用
  * `JsonParser`作为一个光标, 指向正在读取的位置 (准确的说, token). 位置可以是
    * START_OBJECT : `{`
    * END_OBJECT: `}`
    * START_ARRAY: `[`
    * END_ARRAY: `]`
    * 属性名
    * 属性值
  * `JsonParser.nextToken()`移动光标到下一个Token, 并同时返回该token
  * `JsonParser.getCurrentName()`获取当前Token名, 主要用于字段名
  * `JsonParser.getXXXValue()` 读取当前Token, 转化为XXX类型, 主要用于字段值.

* 使用例子

  Json文本

  ```json
  {
      "id":1125687077,
      "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
      "fromUserId":855523, 
      "toUserId":815309,
      "languageCode":"en"
  }
  ```

  对应的实体

  ```java
  @Data
  public class TwitterEntry
  {
      long _id;  
      String _text;
      int _fromUserId, _toUserId;
      String _languageCode;
  }
  ```

  映射到实体的方法

  ```java
  TwitterEntry read(JsonParser jp) throws IOException
  {
      // Sanity check: verify that we got "Json Object":
      if (jp.nextToken() != JsonToken.START_OBJECT) {
          throw new IOException("Expected data to start with an Object");
      }
      TwitterEntry result = new TwitterEntry();
      // Iterate over object fields:
      while (jp.nextToken() != JsonToken.END_OBJECT) {
          String fieldName = jp.getCurrentName();
          // Let's move to value
          jp.nextToken();
          if (fieldName.equals("id")) {
              result.setId(jp.getLongValue());
          } else if (fieldName.equals("text")) {
              result.setText(jp.getText());
          } else if (fieldName.equals("fromUserId")) {
              result.setFromUserId(jp.getIntValue());
          } else if (fieldName.equals("toUserId")) {
              result.setToUserId(jp.getIntValue());
          } else if (fieldName.equals("languageCode")) {
              result.setLanguageCode(jp.getText());
          } else { // ignore, or signal error?
              throw new IOException("Unrecognized field '"+fieldName+"'");
          }
      }
      jp.close(); // important to close both parser and underlying File reader
      return result;
  }
  ```

### 写入

# 四 配置

> 所有注解参考[Jackson Annotations](https://github.com/FasterXML/jackson-docs/wiki/JacksonAnnotations)

## 序列化相关

### 更改属性名

* 方式一

  使用`@JsonProperty`, 仅Serialize时有效
  ```java
  class Student{
      @JsonProperty("Name")
      String name;
      String sex;
  
      /****setter and getter****/
    ...
  }
  ```
  
* 方式二

  Serialize和Deserialize都有效

  ```java
  class Coordinates {
      private int red;
  
      //# Used during serialization
      @JsonGetter("r")
      public int getRed() {
          return red;
      }
  
      //# Used during deserialization
      @JsonSetter("red")
      public void setRed(int red) {
          this.red = red;
      }
  }
  ```
  
  > `@JsonGetter`只能注解
  


### 忽略属性

#### 默认规则

即`DEFAULT_VIEW_INCLUSION`, 表示没有被显式注解的属性都将参与到序列化与解析的过程中.

Jackson可识别的属性:

* 有`getXXX`方法视作存在`XXX`属性
* 共有字段视作存在属性

> 当私有字段存在setter,getter方法时, 注解可直接写到字段上.

可以显示声明属性, 如上面的`@JsonProperty`不传参数; 可显式忽略属性, 如下面的`@JsonIgnore`

#### 静态忽略

即使用时, 将序列化的属性已经固定死了, 用于微调默认行为. 常用的注解如下:

* `@JsonIgnore`可以作用到字段, setter, getter方法上, 都生效, 会忽略属性的**序列化和解析**过程.

  > 官网说`@JsonProperty`可以重置`@JsonIgnore`的读或写. 经测试, 无效!

* `@JsonIgnoreProperties`注解在类上, 功能类似, 除此之外, 能够允许解析时忽略未知的属性.

例子

```java
@JsonIgnoreProperties({ "age", "school" })
class Student{
    String name;
    @JsonIgnore
    String sex;
    int age;
    String school;

	/****setter and getter****/
	...
}
```
#### 动态忽略-view

`@JsonView`可定义好不同的视图, 有不同的序列化和解析规则. 使用时可指定不同的view, 即可实现动态解析.

如何使用?

1. 定义好接口或类, 作为视图的**标志**.

   > 不需要其他成员

2. 在实体类的字段上, 用`@JsonView`配上标志类(接口), 为View指定不同的属性.

3. 使用时, 需要关闭默认规则`DEFAULT_VIEW_INCLUSION`, 然后指定View, 即通过View更改解析与序列化的规则.

   > 在Spring Boot中, 控制器上用到`@JsonView`的地方会自动关闭默认行为.

> 注意
>
> * 关闭默认规则, 则未标注的将不会被序列化或解析. 
> * Spring Boot中其他未使用该注解的控制器还是使用默认规则

除此之外, `@JsonView`的标志类(接口), 还支持继承.

-----

例子:

定义视图

```java
public class Views {
    interface Public {
    }
 
    interface Internal extends Public {
    }
}
```

实体类上指定视图包含的属性

```java
public class Item {
  
    @JsonView(Views.Public.class)
    public int id;
 
    @JsonView(Views.Public.class)
    public String itemName;
 
    @JsonView(Views.Internal.class)
    public String ownerName;
}
```

> 注意, `public`视图只有两个属性; 由于继承, `internal`视图有三个属性

Spring Boot中使用的例子:

```java
@RestController
public class ItemController {

	@Autowired
	private ItemService itemService;

	@JsonView(View.Public.class)
	@RequestMapping("/")
	public List<Item> getAllItems() {
		return itemService.getAll();
	}

	@RequestMapping("/{id}")
	public Item getItem(@PathVariable Long id) {
		return itemService.get(id);
	}
}
```



> 参考
>
> * https://www.baeldung.com/jackson-json-view-annotation
> * https://spring.io/blog/2014/12/02/latest-jackson-integration-improvements-in-spring
>
> - https://github.com/FasterXML/jackson-databind/
> - http://www.cowtowncoder.com/blog/archives/2011/02/entry_443.html

## 反序列化相关

### 忽略未知属性

类上添加`@JsonIgnoreProperties(ignoreUnknown = true)`, 如

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class LKClassData {

    public String description;
    public String primaryKey;

    @Override
    public String toString() {
        return description;
    }
}
```

> 好像默认忽略未知属性的.

## 序列化或解析方式

* `@JsonDeserialize`用于配置某个属性解析的方式
* `@JsonSerialize`用于配置某个属性序列化的方式.

## date相关

* 依赖引入

  ```xml
  <!-- json附加支持jdk8时间类型的包 -->
  <dependency>
      <groupId>com.fasterxml.jackson.datatype</groupId>
      <artifactId>jackson-datatype-jsr310</artifactId>
      <version>2.9.8</version>
  </dependency>
  ```

  该依赖在序列化时需要, 请确保依赖存在.

* 使用

  在POJO的属性上添加如下注解:

  ```java
  //指定序列化和反序列化的时间格式
  @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
  //指定序列化时的转化器
  @JsonSerialize(using = LocalDateTimeSerializer.class)
  //指定反序列化时的转化器
  @JsonDeserialize(using = LocalDateTimeDeserializer.class)
  ```

  > 注意, 序列化和反序列化时都需要`@JsonFormat`注解

  如果在Spring Boot中, 如果未使用`@RequestBody`注解, 即未使用Jackson反序列化时, 则用Spring的`@DateTimeFormat`注解来反序列化:

  ```java
  @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
  ```

>参考：[Jackson Date](https://www.baeldung.com/jackson-serialize-dates)
## 多态支持

> 能简单上手即可, 不作详细介绍.

* 背景

  一个基类, 有多个子类, 已将某个子类序列化. 问题是, 该使用哪个类型反序列化?

  > 如果对象中包含类型信息, 可先转化为Map, 获取该信息, 然后再转一次. 此为下策. 若对象中不含类型信息, 也没有上下文信息呢? 

* 解决方案

  Jackson提供了两个注解来解决. 使用`@JsonTypeInfo`注解在序列化时注入类型信息, `@JsonSubTypes`注解声明JSON中类型信息与Java Class的对应关系, 反序列化时提供基类Class, Jackson会自动处理类型转化.

* 简单Demo

  ```java
  public class SimpleTest {
      @Test
      public void test() throws IOException, InterruptedException {
          ObjectMapper mapper=new ObjectMapper();
          Dog dog = new Dog();
          dog.name="旺财";
          dog.barkVolume=111;
          String s = mapper.writeValueAsString(dog);
          System.out.println(s);
          Animal animal = mapper.readValue(s, Animal.class);
          System.out.println(animal.getClass());
          System.out.println(animal);
          System.out.println(animal.name);
      }
      /**
       * 基类
       */
      @JsonTypeInfo(use=JsonTypeInfo.Id.CLASS)
      static class Animal { // All animals have names, for our demo purposes...
          public String name;
          protected Animal() { }
  
          @Override
          public String toString() {
              return "Animal{" +
                      "name='" + name + '\'' +
                      '}';
          }
      }
      static class Dog extends Animal {
          public double barkVolume; // in decibels
          public Dog() { }
  
          @Override
          public String toString() {
              return "Dog{" +
                      "name='" + name + '\'' +
                      ", barkVolume=" + barkVolume +
                      '}';
          }
      }
      static class Cat extends Animal {
          boolean likesCream;
          public int lives;
          public Cat() { }
      }
  }
  ```

  这里的`@JsonTypeInfo`使用类全限定名作为类型信息, 序列化时默认将信息存储到对象的`@class`属性中.

  这里没有使用`@JsonSubTypes`声明JSON中类型信息与Java Class的对应关系, 因为JSON中的类全限定名提供的信息已经足够了.

* 复杂一点的Demo

  ```java
      ...
          
   	@JsonTypeInfo(
              use=JsonTypeInfo.Id.NAME, //默认使用类型名的, 如Dog. 但可被@JsonSubTypes提供的name属性更改
              include = JsonTypeInfo.As.PROPERTY,//信息存在属性中, 默认可选
              property = "type"//属性名, 默认@type
      )
      @JsonSubTypes({ //注册Java对象与其JSON中类型信息的对应关系
              @JsonSubTypes.Type(value = Dog.class, name = "dog") ,
              @JsonSubTypes.Type(value = Cat.class, name = "cat")
      })
      static class Animal { // All animals have names, for our demo purposes...
          public String name;
          protected Animal() { }
  
  		...
      }
  	
  	...
  ```

  > 一切尽在注释中

  且外, 即使`@JsonTypeInfo`中`property`指定的属性与对象的属性重名了, 仍会再次插入该属性. 貌似这是个bug.

> 参考:
>
> * [Jackson多态支持](https://www.jianshu.com/p/43564c096f53) 入门实例
> * [十分钟学习Jackson多态处理](https://www.jianshu.com/p/6e3888998b6f) 注解详细使用介绍
>
> * [Type handling](https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations#type-handling) 官方文档

# 五 XML

**jackson-dataformat-xml**项目是Jackson JSON处理器的扩展，用于在java对象与XML之间转换。

使用之前先配置Maven依赖：
```xml
        <!--xml的包，jackson的扩展-->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
            <version>2.9.7</version>
        </dependency>
        <!--jackson.dataformat用到的依赖-->
        <dependency>
            <groupId>com.fasterxml.woodstox</groupId>
            <artifactId>woodstox-core</artifactId>
            <version>5.2.0</version>
        </dependency>
```
基本使用与Jackson JSON的**Data Binding**基本无异，如：
```java
// Important: create XmlMapper; it will use proper factories, workarounds
ObjectMapper xmlMapper = new XmlMapper();
//serializing POJOs as XML
String xml = xmlMapper.writeValueAsString(new Simple());
// or
xmlMapper.writeValue(new File("/tmp/stuff.json"), new Simple());
//Deserializing POJOs from XML
ObjectMapper xmlMapper = new XmlMapper();
Simple value = xmlMapper.readValue("<Simple><x>1</x><y>2</y></Simple>", Simple.class);
```
除了支持Jackson JSON的注解，还支持部分JAXB和该项目自己的注解。详细请参考链接。

-------------
值得了解的xml注解：
* `@JacksonXmlRootElement` allows specifying XML element to use for wrapping the root element (default uses 'simple name' of the value class)
* `@JacksonXmlCData` allows specifying that the value of a property is to be serialized within a CData tag.

# 踩坑

## 属性名变小写了

属性名在序列化后会改变, 改变规则如下:

> 第一个或第一个字符是大写时, 该字符及其之后**连续**的大写字符都将转化成小写

例子: 

```
pId -> pid
Pid -> pid
PId -> pid
PIIIDDDDD -> piiiddddd
ssId -> ssId
sIDDDDD -> siddddd
```

## 解决BigDecimal精度丢失问题

```java
objectMapper.setNodeFactory(JsonNodeFactory.withExactBigDecimals(true));
objectMapper.configure(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS, true);
```

> 参考[Deserializing BigDecimal using JsonNode loses precision](https://github.com/FasterXML/jackson-databind/issues/2087#issuecomment-593670525)

# 参考
* [FasterXML/jackson-databind](https://github.com/FasterXML/jackson-databind/) Json序列化的官方教程
* [Jackson - Data Binding](https://www.tutorialspoint.com/jackson/jackson_data_binding.htm) tutorialspoint上的简易教程
* [FasterXML/jackson-dataformat-xml](https://github.com/FasterXML/jackson-dataformat-xml) xml序列化的官方教程
* [Jackson Annotations](https://github.com/FasterXML/jackson-docs/wiki/JacksonAnnotations) Json所有注解

---------

进阶阅读

- [Overall Jackson Docs](https://github.com/FasterXML/jackson-docs) 全面的文档
- [Project wiki page](https://github.com/FasterXML/jackson-databind/wiki)