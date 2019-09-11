[TOC]

# 一 介绍
jackson用于在java对象与JSON之间映射。jackson有三种处理JSON的处理模型：
* **Data Binding**：在JSON与简单对象（POJO，Maps, Lists, Strings, Numbers, Booleans 和null）或更复杂一点的对象（如对象中还有其他复杂对象）之间转换。
* **Tree Model**：在内存中，JSON被表示为树的形式，便于遍历。
* **Streaming Model**：前面两种处理模型都是基于该模型的，使用它主要是为了获得更高的性能或更细致的控制。

这里讲述Data Binding模型。

# 二 Maven配置
```xml
      <dependency>
          <groupId>com.fasterxml.jackson.core</groupId>
          <artifactId>jackson-core</artifactId>
          <version>2.9.7</version>
      </dependency>
      <dependency>
          <groupId>com.fasterxml.jackson.core</groupId>
          <artifactId>jackson-annotations</artifactId>
          <version>2.9.7</version>
      </dependency>
      <dependency>
          <groupId>com.fasterxml.jackson.core</groupId>
          <artifactId>jackson-databind</artifactId>
          <version>2.9.7</version>
      </dependency>
```
# 三 使用
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

## POJO与JSON
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
## Map,List与JSON
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
# 四 注解
## 更改属性名
使用`@JsonProperty`：
```java
class Student{
    @JsonProperty("Name")
    String name;
    String sex;
    
	/****setter and getter****/
	...
}
```
## 忽略属性

### 默认规则

序列化规则:

* 有`getXXX`方法视作存在`XXX`属性
* 共有字段视作存在属性

当私有字段存在setter,getter方法时, 注解可直接写到字段上.

### 简单方式

* `@JsonIgnore`可以作用到字段, setter, getter方法上, 都生效, 会忽略属性的**读和写**.

  > * 官网说`@JsonProperty`可以重置`@JsonIgnore`的读或写. 经测试, 无效!

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
### View

未完待续

* https://github.com/FasterXML/jackson-databind/
* http://www.cowtowncoder.com/blog/archives/2011/02/entry_443.html

## date相关

* 序列化`LocalDateTime`
	需要先引入包
	```xml
	      <!-- json附加支持jdk8时间类型的包 -->
	      <dependency>
	          <groupId>com.fasterxml.jackson.datatype</groupId>
	          <artifactId>jackson-datatype-jsr310</artifactId>
	          <version>2.9.8</version>
	      </dependency>
	```
	然后在属性上添加注解
	```java
		@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
		@JsonSerialize(using=LocalDateTimeSerializer.class)
	```
* 解序列化，这里暂时使用spring的注解
	```java
		@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
	```
>以后参考：[Jackson Date](https://www.baeldung.com/jackson-serialize-dates)
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

# 参考
[FasterXML/jackson-databind](https://github.com/FasterXML/jackson-databind/)
[Jackson - Data Binding](https://www.tutorialspoint.com/jackson/jackson_data_binding.htm)
[FasterXML/jackson-dataformat-xml](https://github.com/FasterXML/jackson-dataformat-xml)