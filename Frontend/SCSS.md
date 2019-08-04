[TOC]

# 一 介绍

* scss最终会被编译成css。scss允许使用变量、嵌套语法、混合、函数等特性，并且兼容css语法。
* scss其实只是sass的一种语法形式（css超集，后缀`.scss`），sass还有缩进语法（后缀`.sass`）。
* 默认编码`utf-8`
* 但scss语法错误时，scss会直接解析失败，与css不同。
* scss扩展了[语句和表达式](<https://sass-lang.com/documentation/syntax/structure>)
* 注解
  * `//`：单行注解，不会编译成css
  * `/* */`：多行注解，里面的插值会被计算。编译成css，除了压缩模式。压缩模式中使用`/*! */`。
  * 文档注解：略
* [特殊函数](<https://sass-lang.com/documentation/syntax/special-functions>)：一些css的本地函数与scss冲突，因此会被处理后才转化css本地函数。
>发现，scss并不是完全兼容css，如`@import "//at.alicdn.com/t/font_1199749_t172xat03rn.css"`就错误了，需要使用`url()`
# 二 样式规则

## 2.1 嵌套

### 2.1.1 普通嵌套（nesting）

```scss
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```

转化为css：

```css
nav ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
nav li {
  display: inline-block;
}
nav a {
  display: block;
  padding: 6px 12px;
  text-decoration: none;
}

```

> 注意，不要嵌套太深。否则会降低性能，和不直观

### 2.1.2 嵌套列表

```scss
.alert, .warning {
  ul, p {
    margin-right: 0;
    margin-left: 0;
    padding-bottom: 0;
  }
}
```

css中为两者的笛卡尔积形式：

```css
.alert ul, .alert p, .warning ul, .warning p {
  margin-right: 0;
  margin-left: 0;
  padding-bottom: 0;
}
```

### 2.1.3 嵌套组合（combinators）

combinators的放置位置有三种

```scss
ul > {
  li {
    list-style-type: none;
  }
}

h2 {
  + p {
    border-top: 1px solid gray;
  }
}

p {
  ~ {
    span {
      opacity: 0.8;
    }
  }
}
```

转化为css：

```css
ul > li {
  list-style-type: none;
}

h2 + p {
  border-top: 1px solid gray;
}

p ~ span {
  opacity: 0.8;
}
```

### 2.1.4 父类选择器`&`

用于添加伪类或在`&`前添加选择器。

```scss
.alert {
  // The parent selector can be used to add pseudo-classes to the outer
  // selector.
  &:hover {
    font-weight: bold;
  }

  // It can also be used to style the outer selector in a certain context, such
  // as a body set to use a right-to-left language.
  [dir=rtl] & {
    margin-left: 0;
    margin-right: 10px;
  }

  // You can even use it as an argument to pseudo-class selectors.
  :not(&) {
    opacity: 0.8;
  }
}
```

```css
.alert:hover {
  font-weight: bold;
}
[dir=rtl] .alert {
  margin-left: 0;
  margin-right: 10px;
}
:not(.alert) {
  opacity: 0.8;
}

```

-------

还能添加后缀

```scss
.accordion {
  max-width: 600px;
  margin: 4rem auto;
  width: 90%;
  font-family: "Raleway", sans-serif;
  background: #f4f4f4;

  &__copy {
    display: none;
    padding: 1rem 1.5rem 2rem 1.5rem;
    color: gray;
    line-height: 1.6;
    font-size: 14px;
    font-weight: 500;

    &--open {
      display: block;
    }
  }
}
```

结果为

```css
.accordion {
  max-width: 600px;
  margin: 4rem auto;
  width: 90%;
  font-family: "Raleway", sans-serif;
  background: #f4f4f4;
}
.accordion__copy {
  display: none;
  padding: 1rem 1.5rem 2rem 1.5rem;
  color: gray;
  line-height: 1.6;
  font-size: 14px;
  font-weight: 500;
}
.accordion__copy--open {
  display: block;
}
```



## 2.2 插值

表达式可以用在属性值、函数、甚至类名上。

```scss
@mixin define-emoji($name, $glyph) {
  span.emoji-#{$name} {
    font-family: IconFont;
    font-variant: normal;
    font-weight: normal;
    content: $glyph;
  }
}

@include define-emoji("women-holding-hands", "👭");
```

```scss
.circle {
  $size: 100px;
  width: $size;
  height: $size;
  border-radius: $size / 2;
}
```

# 参考

* [Documentation](<https://sass-lang.com/documentation>)	