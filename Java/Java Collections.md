# 一 介绍

* 存储元素的容器, 用于存储,访问,操纵和聚合数据.

* 组成
  * 接口
  * 实现
  * 算法


# 二 接口

* 层次结构

	![Two interface trees, one starting with Collection and including Set, SortedSet, List, and Queue, and the other starting with Map and including SortedMap.](.Java%20Collections/colls-coreInterfaces.gif) 

* 约定: 接口中的修改方法不一定必须得实现, 而是可选的. 如果访问未实现方法, 将抛出  [`UnsupportedOperationException`](https://docs.oracle.com/javase/8/docs/api/java/lang/UnsupportedOperationException.html)异常.

## 概述

* `Collection` 集合的根接口, 拥有最通用的操作方法

* `Set` 非重复元素的集合

* `List` 含有序, 可重复元素的集合

* `Queue` 存储待处理元素的队列集合. 允许元素队头进入, 队尾出去.

  > 那个元素先出去, 与其元素的顺序有关, 如以优先级为顺序, 优先级高的先出去. 一般是先进先出的模式.

* `Deque` 储存待处理元素的双端队列集合. 两端都允许插入或删除.

* `Map` 映射`Key`到`Value`的对象, `Key`不能重复

最后两个接口是`Set`和`Map`的有序版本

* `SortedSet` 维护元素顺序的`Set`
* `SortedMap` 通过`Key`来维护元素顺序的`Map`

>**栈呢**? 栈拥有先进后出的特性, 是Deque的子集, 可由其实现类`LinkedList`代替

> 关于有序
>
> * **不管你怎么存储, 只要遍历时有序即可**, 遍历时将以升序的方式遍历
> * 集合中的元素顺序, 可由元素的自然顺序 ([natural ordering](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Comparable.html)) 或[`Comparator`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html) 给出. 
> * 有序的`Set`和`Map`在创建时就固定了其排序方法 (上述两种之一) ; 而`List`创建之后, 一般按照先后顺序排序, 也可以由`Collections`为其重新排序

## Collection

集合的基本操作:

* **转化**

  * 为构造函数: 让一个新的集合从任意类型的集合中构造出来. 如

      ```java
      List<String> list = new ArrayList<>(c);
      ```

  * 为数组` toArray() `

* **添加**元素或集合

  `add`,`addAll`

* **删除**

  `remove`系列, `clear()`

* 获取**流**

  `steam()`,`parallelStream()`

* **查找**

  * 查找元素位置 [indexOf](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#indexOf(java.lang.Object)) 
  * 是否包含元素 [contains(Object o)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#contains(java.lang.Object)) 

  > 使用`Objects.equals`判断元素是否相等

* **遍历**

  * 使用流(推荐)

  * `for-each`, 即增强型`for`循环, 见[Iterable](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Iterable.html)

  * 获取遍历器`iterator()`, 支持遍历过程中删除( `remove()` )当前元素.

    > 每调用一次`next()`, 最多执行一次删除, 否则会报异常.

  > 何时使用`iterator` ?
  >
  > * 想删除当前元素时, 其他方式可能会出现问题
  > * 想同时遍历多个集合

* **常用算法**

   `Collections`提供更多操纵集合的静态方法

## Set

* 实现类

  *  `HashSet` 以hash表的方式存储数据, 性能最佳, 但不保证访问有序

  * `TreeSet` 存储数据到红黑树中, 有序存储, 性能略差

  * `LinkedHashSet` 是`HashSet`的有序版本, 维护一个双向链表, 记录插入顺序.

    > 本质上就是在节点上添加了指向前继,后继节点的指针, 其他的都是`Hashset`的功能, 只不过多了维护双向链表的逻辑
    >
  
> 查看其源码, 会发现, `Set`都是通过对应`Map`来实现的, 将`Map`的`Value`置为了一个固定的对象.

* 常用于集合间去重, `LinkedHashSet`可以在去重的同时保留其他排序关系

## List

* 除了集合通用操作, 还有
  * ` Positional access`直接操纵某个位置上的元素
  * 搜索指定元素
  * 更具体的遍历器
  * 范围操作
  * 排序`sort()`
* 实现类
  *  `ArrayList` 数组, 适合访问元素较多的情况
  * `LinkedList` 链表, 适合增删元素较多的情况

## 队列

### Queue

* 操纵方法: 有两种形式

  | ype of Operation | Throws exception | Returns special value |
  | ---------------- | ---------------- | --------------------- |
  | Insert           | `add(e)`         | `offer(e)`            |
  | Remove           | `remove()`       | `poll()`              |
  | Examine          | `element()`      | `peek()`              |

  > 操作失败时的提示方式不同.

* 实现类
  
  * `LinkedList`

### Deque

* 实现类
  * `ArrayDeque`
  * `LinkedList`

* 操纵方法: 同样两种形式

  | Type of Operation | First Element (Beginning of the `Deque` instance) | Last Element (End of the `Deque` instance) |
  | ----------------- | ------------------------------------------------- | ------------------------------------------ |
  | **Insert**        | `addFirst(e)` `offerFirst(e)`                     | `addLast(e)` `offerLast(e)`                |
  | **Remove**        | `removeFirst()` `pollFirst()`                     | `removeLast()` `pollLast()`                |
  | **Examine**       | `getFirst()` `peekFirst()`                        | `getLast()` `peekLast()`                   |

## Map

* 实现类
  
  > 与`Set`类似, 略
  
*  [`HashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) 
  * [`TreeMap`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html)
  * [`LinkedHashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) 
  
* 集合视图
  * `keySet` — the `Set` of keys contained in the `Map`.
  * `values` — The `Collection` of values contained in the `Map`. This `Collection` is not a `Set`, because multiple keys can map to the same value.
  * `entrySet` — the `Set` of key-value pairs contained in the `Map`. The `Map` interface provides a small nested interface called `Map.Entry`, the type of the elements in this `Set`.
  
* 实用方法

  * `put()` 设置键值对
  * `get()` 获取值
  * `V getOrDefault(Object key, V defaultValue)` 获取值, 若获取不到, 则取`defaultValue`

* 其他

  * ~~元素相等, 当且仅当`Key`的`equals`和`hashcode`值一致~~

# 三 流

## 介绍

A *stream* is a sequence of elements. Unlike a collection, it is not a data structure that stores elements. Instead, a stream carries values from a source through a pipeline 

> 简而言之, 流就是一个流经管道的元素流, 在管道的每个节点 (中间操作) 上加工处理, 最终得到一个结果 (结束操作) .

## 区别于集合的特性

* **No storage**. A stream is not a data structure that stores elements; **instead, it conveys elements from a source** such as a data structure, an array, a generator function, or an I/O channel, **through a pipeline of computational operations.**
* **Functional in nature**. An operation on a stream produces a result, **but does not modify its source**. For example, filtering a `Stream` obtained from a collection produces a new `Stream` without the filtered elements, rather than removing elements from the source collection.
* **Laziness-seeking.** Many stream operations, such as filtering, mapping, or duplicate removal, can be implemented lazily, exposing opportunities for optimization. For example, "find the first `String` with three consecutive vowels" need not examine all the input strings. Stream operations are divided into intermediate (`Stream`-producing) operations and terminal (value- or side-effect-producing) operations. **Intermediate operations are always lazy.**
* **Possibly unbounded.** While collections have a finite size, streams need not. Short-circuiting operations such as `limit(n)` or `findFirst()` can **allow computations on infinite streams to complete in finite time.**
* **Consumable**. The elements of a stream are only visited once during the life of a stream. Like an [`Iterator`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html), a new stream must be generated to revisit the same elements of the source.

## 组成

* 一个**数据源**: 可以是集合, 数组,  generator 函数或 IO 通道

* 0到多个**中间操作**, 返回一个新的流. 

  > 如`filter()`过滤, 得到新的流; `map()`将流映射为其他形式的流; 等等
  >
  
* 一个**结束操作**: 不返回流

  > 如`forEach()`,仅处理流, 不产生新的流.

>* 中间操作是Lazy的, 只有执行结束操作时才执行;
>* 并且不是所有的元素都会被用到, 如`findFirst()`找到第一个流就结束了. 避免性能浪费.

## 操作

* 中间操作
  * 过滤`filter()`
  * 映射`map()`
  * 遍历`peek()`

* 结束操作

  * Reduction : 将流转化为一个结果

    * `reduce()`对每个元素进行同一个操作, 并累积结果, 生成一个新值.

      > 还有其他变种方法, 如`min()`,`max()`,`average()`,`sum()`等

    * `collect()`修改和改变现有的值, 得到一个~~容器~~

      > 使用有点小复杂, 因此提供了 [Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) 类, 含有提供常用聚合操作的静态方法.

  * 遍历`forEach()`
  
  * 获取元素
  
    * `findFirst()`找到流中第一个元素
    * `findAny()`流中找任意一个
  
  * 判断是否匹配
  
    * `anyMatch()`
    * `allMatch()`
    * `noneMatch()`

## 平行流

将流分成子流, 每个子流在单独的线程中执行所有操作, 然后再组合所有子流的结果.

> 在数据量大的情况下效率会高点, 但要考虑多线程问题. 见[Parallelism](https://docs.oracle.com/javase/tutorial/collections/streams/parallelism.html)

# 四 实现

集合接口的实现有很多中:

- **General-purpose implementations** are the most commonly used implementations, designed for everyday use. They are summarized in the table titled General-purpose-implementations.
- **Special-purpose implementations** are designed for use in special situations and display nonstandard performance characteristics, usage restrictions, or behavior.
- **Concurrent implementations** are designed to support high concurrency, typically at the expense of single-threaded performance. These implementations are part of the `java.util.concurrent` package.
- **Wrapper implementations** are used in combination with other types of implementations, often the general-purpose ones, to provide added or restricted functionality.
- **Convenience implementations** are mini-implementations, typically made available via static factory methods, that provide convenient, efficient alternatives to general-purpose implementations for special collections (for example, singleton sets).
- **Abstract implementations** are skeletal implementations that facilitate the construction of custom implementations — described later in the [Custom Collection Implementations](https://docs.oracle.com/javase/tutorial/collections/custom-implementations/index.html) section. An advanced topic, it's not particularly difficult, but relatively few people will need to do it.

其中通用实现如下

| Interfaces | Hash table Implementations | Resizable array Implementations | Tree Implementations | Linked list Implementations | Hash table + Linked list Implementations |
| ---------- | -------------------------- | ------------------------------- | -------------------- | --------------------------- | ---------------------------------------- |
| `Set`      | `HashSet`                  |                                 | `TreeSet`            |                             | `LinkedHashSet`                          |
| `List`     |                            | `ArrayList`                     |                      | `LinkedList`                |                                          |
| `Queue`    |                            |                                 |                      |                             |                                          |
| `Deque`    |                            | `ArrayDeque`                    |                      | `LinkedList`                |                                          |
| `Map`      | `HashMap`                  |                                 | `TreeMap`            |                             | `LinkedHashMap`                          |

> 参考[Implementations](https://docs.oracle.com/javase/tutorial/collections/implementations/index.html)

# 五 算法

集合的所有算法都在 [`Collections`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)  类中以静态方法提供, 可分为六类

- [Sorting](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html#sorting)
- [Shuffling](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html#shuffling)
- [Routine Data Manipulation](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html#rdm)
- [Searching](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html#searching)
- [Composition](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html#composition)
- [Finding Extreme Values](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html#fev)

> 参考[Algorithms](https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html)

# 其他

## hash & hashCode() & equals()

在hash表中，如`HashTable`、`HashMap`或`HashSet`中插入一元素时，先利用`hashCode()`方法得到hash值，然后经过hash映射, 得到元素在hash表内部的存放位置. 不同元素可能会对应相同hash值，因此可能产生冲突，利用`equals()`判断相同位置的元素是否为同一元素，然后放入不同位置。

因此要确定元素在hash表中的位置, 至少用到`hashCode()`和`equals()`函数. 在使用hash表存储复杂对象时，最好重新实现这两个方法.

## 元素顺序

集合中元素的顺序可由以下两者之一确定:

*  *natural ordering* : 元素的自然顺序由`Comparable`接口确定
*  `Comparator` : 排序器

> 基本类型的包装类都有自己的自然顺序规则, 如2大于1

遍历时, 将以升序的方式遍历

主要内容见1.1概述

# 参考

* [oracle tutorial](https://docs.oracle.com/javase/tutorial/collections/TOC.html)
* [Stream API](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#peek(java.util.function.Consumer))
* [package info](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html)



















