# 开始

* 介绍

  * 提供解析html、操作DOM的api，类似jquery的方法。
  * 自动校正html中错误语法。甚至只有`div`元素，都会被补充成完整完整。

* 引入maven项目：

  ```xml
  <dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.12.1</version>
  </dependency>
  ```

# 使用

* `Jsoup`：通过该类，可传入url、字符串、输入流、文件的方式输入html，解析后会返回`Document`表示html文档，然后进行DOM操作。

   * `Jsoup.parse(String html)`：从字符串中获取html并解析

   * `Jsoup.parse(String html,String baseUri)`：`baseUri`指定html中相对地址的基址，当获取相对地址的绝对地址时有用。如果html中含有`base`元素时或不需要该功能，可不用该方法。

   * `Jsoup.connect(String url)`：从url上获取，例子如下：

     ```java
     Document doc = Jsoup.connect("http://example.com/").get();
     String title = doc.title();
     //更复杂的例子
     Document doc = Jsoup.connect("http://example.com")
       .data("query", "Java")
       .userAgent("Mozilla")
       .cookie("auth", "token")
       .timeout(3000)
       .post();
     ```

     > 这种方式已默认设置好了baseurl

  * `Jsoup.parse(File in, String charsetName, String baseUri)`：从文件中获取html

* 查找元素：[DOM方式](<https://jsoup.org/cookbook/extracting-data/dom-navigation>)和[css选择器方式](<https://jsoup.org/cookbook/extracting-data/selector-syntax>)

* 获取数据
  * [Use DOM methods to navigate a document](<https://jsoup.org/cookbook/extracting-data/dom-navigation>)
  * [Extract attributes, text, and HTML from elements](<https://jsoup.org/cookbook/extracting-data/attributes-text-html>)

* 获取URL：`Element`类的`attr("href")`方法获取url；加上前缀`abs`后，会结合之前设置的`baseUri`解析成绝对地址，如`attr("abs:href")`。如果没有设置则返回空。

* 修改DOM和其他内容，见参考文献

# 参考

[jsoup cookbook](<https://jsoup.org/cookbook/>)