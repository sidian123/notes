[TOC]

# 一 介绍
java 8提供了java.time包来表示、操作时间。主要涉及到了date、time、instant、duration、timezone的概念。这些时间类都是基于ISO日历系统的。这些时间类都是不可变的，因此也是线程安全的。

在java8之前，时间类有java.util下的Date、Calendar等类提供。但是由于它们类型不安全、线程不安全（可变的）、易出bug（比如不常用的月份编号方式）等问题，导致这些遗留类不在常用。

## 时间表示
有两种表示时间的方式：
* **human time**：以人类可读的方式表示时间，如含有一些如下字段：year、month、day、hour、minute、second、offset和zone id。前面三个字段用**Date**表示，接下来三个用**Time**表示，最后两个和**时区**有关。
* **machine time**：以从`1970-01-01T00:00:00Z`（即`epoch`，Z表示零时区，即UTC时间）开始到现在的秒数来表示。

## offset和zone id
每个时区（time zone）都有一个地方时（local time），但通常以零时区的地方时UTC作为标准来换算, 两者的时差以offset(对应类`ZoneOffset`)来表示, 如东八区offset为`UTC+8：00`.

然而, 有些地区采用夏令时, 即夏季时, 当地地方时要在原有的基础上加一. 因此即使是同一地区, 在不同的时间段也会有不同的地方时, offset也不同. 因此Java提供了`ZoneId`类来标识时区, 如`Europe/Paris`; `Zoneid`内部含有`ZoneRules`类, 它记录了历史上和未来的时区（time zone）与offset的变化关系信息，包括了夏令时信息。

>可以看出，**通过zone id可以得到offset**，但是offset不能得到zone id。但要注意，zone id并不一定能得到全面的offset信息，因此可以自己提供`ZoneRules`的相关实现

machine time本身是以现在到epoch之间的时间为度量的，epoch本身就以UTC表示的。因此主要是human time相关的类需要时区信息，有些类不含时区信息，但是这些类的对象一般以默认时区通过系统时钟获得（如`now`函数）。

预备知识：[时区(time zone)和地方时(local time)][1]

## 类与字段
每个类与字段的关系如下：

|Class or Enum|Year|Month|Day|Hours|Minutes|Seconds*|Zone Offset|Zone ID|toString Output|
|--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |
|Instant||||||✔|||2013-08-20T15:16:26.355Z|
|LocalDate|✔|✔|✔| | | | | | 2013-08-20 |
|LocalDateTime|✔|✔|✔|✔|✔|✔|||2013-08-20T08:16:26.937|
|ZonedDateTime|✔|✔|✔|✔|✔|✔|✔|✔|2013-08-21T00:16:26.941+09:00[Asia/Tokyo]|
|LocalTime||||✔|✔|✔|||08:16:26.943|
|MonthDay||✔|✔||||||--08-20|
|Year|✔||||||||2013|
|YearMonth|✔|✔|||||||2013-08|
|Month||✔|||||||AUGUST|
|OffsetDateTime|✔|✔|✔|✔|✔|✔|✔||2013-08-20T08:16:26.954-07:00|
|OffsetTime||||✔|✔|✔|✔||08:16:26.957-07:00|
|Duration|||**|**|**|✔|||PT20H (20 hours)|
|Period|✔|✔|✔||||***|***|P10D (10 days)|

\* Seconds are captured to nanosecond precision.
\*\* This class does not store this information, but has methods to provide time in these units.
\*\*\* When a Period is added to a ZonedDateTime, daylight saving time or other local time differences are observed.见2.2

# 二 详细介绍
主要参考：[Package java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)
## 常用的类
* `Instant` ：表示时间戳，也就是上面讲的machine time。当前的Instant可以通过`Clock`类获得。
* `LocalDate`： stores a date without a time. 比如日期：'2010-12-03'
* `LocalTime`：stores a time without a date. 比如'11:30' 
* `LocalDateTime`：stores a date and time. 比如 '2010-12-03T11:30'.
* `OffsetTime`：stores a time and offset from UTC without a date.比如'11:30+01:00'
* `OffsetDateTime`：stores a date and time and offset from UTC.比如'2010-12-03T11:30+01:00'
* `ZonedDateTime`：stores a date and time with a time-zone. （其实是存储zone id，但ZoneId可以得到offset）

使用带有zone id的ZonedDateTime会给应用带来复杂性，因此一般推荐使用local类型的类。而OffsetDateTime只有offset，适合于在网络上和数据库中传输数据。

## Duration和Perid
Duration和Period都是度量两个时间之间的时差，都是通过静态方法`between`生成，区别如下：
* Duration：使用time-base的值描述时差（即seconds，nanoseconds）。可以为负。添加到ZonedDateTime中，即使存在夏令时，也会产生正确的结果。
* Perid：使用date-base的值描述时差（即years，months，days）。添加到ZonedDateTime中时，如果存在夏令时，结果可能会出错，因为它是简单的加到对应字段中。

## 其他类
* `Month` stores a month on its own. This stores a single month-of-year in isolation, such as 'DECEMBER'.
* `DayOfWeek` stores a day-of-week on its own. This stores a single day-of-week in isolation, such as 'TUESDAY'.
* `Year` stores a year on its own. This stores a single year in isolation, such as '2010'.
* `YearMonth` stores a year and month without a day or time. This stores a year and month, such as '2010-12' and could be used for a credit card expiry.
* `MonthDay` stores a month and day without a year or time. This stores a month and day-of-month, such as '--12-03' and could be used to store an annual event like a birthday without storing the year.

## 方法
时间类的方法名都统一命名且有规律，如下是通用的方法名或前缀：
* `of` - static factory method
* `parse` - static factory method focussed on parsing
* `get` - gets the value of something
is - checks if something is true
* `with` - the immutable equivalent of a setter
* `plus` - adds an amount to an object
* `minus` - subtracts an amount from an object
* `to` - converts this object to another type
* `at` - combines this object with another, such as date.atTime(time)
* `parse` - Parses the input string to produce an instance of the target class.
* `format` - Uses the specified formatter to format the values in the temporal object to produce a string.
* `now` - Obtains the current time from the system clock in the default time-zone

>注意点
>* 时间类不通过构造函数生成对象，而是通过`of`工厂方法
>* 实际上，一般都是使用`now`来生成时间类的，该函数通过`System.currentTimeMillis()`获得毫秒。
>* 由于时间类都是不可变的，因此没有set方法，而是通过`with`方法返回对应字段修改后的副本。

还有一些其他辅助包下的类，它们主要用于时间的计算，比如`TemporalAdjuster`类。这里不讲了。

# 三 其他
## 遗留代码
java8之前，时间类是由`java.util.Date`, `java.util.Calendar`和`java.util.TimeZone` classes, 以及它的子类，比如`java.util.GregorianCalendar.` 

其中，`Instant` 和 `java.util.Date`类似；`ZonedDateTime` 和`java.util.GregorianCalendar`类似；`java.util.TimeZone`相当于`java.time.ZoneId`或`java.time.ZoneOffset`

他们之间的转化关系如下：
* `Calendar.toInstant()` converts the Calendar object to an Instant.
* `GregorianCalendar.toZonedDateTime() `converts a GregorianCalendar instance to a ZonedDateTime.
* `GregorianCalendar.from(ZonedDateTime) `creates a GregorianCalendar object using the default locale from a ZonedDateTime instance.
* `Date.from(Instant)` creates a Date object from an Instant.
* `Date.toInstant()` converts a Date object to an Instant.
* `TimeZone.toZoneId()` converts a TimeZone object to a ZoneId.

## DateTimeFormatter
时间类都有parse和format方法，分别用来解析字符串、生成字符串，都用到了DateTimeFormatter作为参数。该类用作时间格式器，格式化时间。有三种方法配置格式器：
1. 使用预定义的格式器，如ISO_LOCAL_DATE。DateTimeFormatter提供了一系列的该类型实例的静态变量。
2. 使用默认locale，比如解析输出中文啥滴；然后设置locale样式，比如long或medium。使用`ofLocal...`方法产生实例。
3. 使用模式字符（pattern letters），如uuuu-MMM-dd。使用`ofPattern`方法产生实例。

这里详细介绍第三种方式。所有字母都保留，用作模式字符，标点符号不是。一些有用的符号如下：

| Symbol | Meaning                     | Presentation | Examples                       |
| ------ | :-------------------------- | :----------- | :----------------------------- |
| u      | year                        | year         | 2004; 04                       |
| y      | year-of-era                 | year         | 2004; 04                       |
| D      | day-of-year                 | number       | 189                            |
| M/L    | month-of-year               | number/text  | 7; 07; Jul; July; J            |
| d      | day-of-month                | number       | 10                             |
| H      | hour-of-day (0-23)          | number       | 0                              |
| m      | minute-of-hour              | number       | 30                             |
| s      | second-of-minute            | number       | 55                             |
| V      | time-zone ID                | zone-id      | America/Los_Angeles; Z; -08:30 |
| '      | escape for text，用来转义的 | delimiter    |                                |

>DateTimeFormatter也是不可变的，线程安全的。

参考：[java.time.format.DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)

## locale vs timezone
locale是一组参数，它定义用户的语言、区域和用户希望在用户界面中看到的任何特殊变体首选项；而timezone是地球上为法律、商业和社会目的遵守统一标准时间的区域。

我之前把两者弄混淆了，它们两个是不同的东西。locale主要是用在用户界面中设置语言地区相关的选项，而timezone是时区，和时间有关。比如美国只有一个locale（en-US），但是有多个时区，不同地方可能地方时不同。

参考：
[How to get the current Time and TimeZone from Locale?](https://stackoverflow.com/questions/10570884/how-to-get-the-current-time-and-timezone-from-locale)
[Locale (computer software)](https://en.wikipedia.org/wiki/Locale_(computer_software))
[Language localisation](https://en.wikipedia.org/wiki/Language_localisation)
[Time zone](https://en.wikipedia.org/wiki/Time_zone)

# 参考
[时区(time zone)和地方时(local time][1]
[Package java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)：api文档，下面的总结很详细
[Date Time tutorial](https://docs.oracle.com/javase/tutorial/datetime/TOC.html)


[1]:https://blog.csdn.net/jdbdh/article/details/83211809