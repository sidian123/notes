# 介绍

一个注解驱动的, 用于进行单元测试的框架. 

有两种测试方法:

1. 手动测试

   因此IDE工具的支持, 直接手动运行单个测试用例即可

2. 自动测试

   需要工具支持, 如Maven, 在编译时会进行自动测试.

   > 即使是Maven项目, 在Idea中运行时, 也不会自动测试, 因为idea它接管了Maven的部分功能.

依赖

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.5.2</version>
    <scope>test</scope>
</dependency>
```

# API使用

## 测试方法相关

* `@Test`: 标注方法为**测试方法**. 其中`timeout`参数指定失败时间
* `@BeforeClass`: 所有测试方法开始前调用, 且**仅一次**

* `@Before`: 每个测试方法前**都调用**
* `@AfterClass`: 所有测试方法结束后调用, 且**仅一次**
* `@After`: 每个测试方法结束后**都调用**

## 断言相关

> 搞做了, 这是`assertj-core`的内容

即在某处断言某个结果, 全部断言成功则测试成功, 否则失败.

`Assertions`类常用的静态方法如下:

* `void assertEquals(boolean expected,boolean actual)`: checks that two primitives/objects are equal. It is overloaded.

* `void assertTrue(boolean condition)`: checks that a condition is true.

* `void assertFalse(boolean condition)`: checks that a condition is false.

* `void assertNull(Object obj)`: checks that object is null.

* `void assertNotNull(Object obj)`: checks that object is not null.

除此之外, 测试函数不抛出异常也是算测试成功的.

