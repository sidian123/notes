* Java技术体系

  * Java技术体系组成

    Java语言、JVM、Class字节码、Java API、各种第三方框架。

  * JDK（Java Development Kit）

    Java语言、Jvm、Java API。JDK是用于支持java程序开发的最小环境。

  * JRE（Java Runtime Environment）

    Java SE API、JVM。JRE是支持java程序运行的标准环境。

  * Java技术体系可分为4个平台，包含的java api内容有大有小：
    * Java Card，运行在小内存设备上；
    * Java ME（Micro Edition），支持移动终端，java api有所精简；
    * Java SE（Standard Edition），支持桌面级应用，含有完整的java核心api；
    * Java EE（Enterprise Edition），对java se api做了大量扩充，扩展api一般以javax.*作为包名。

* 内存模型

  * 堆：所有线程共享，存放对象实例。
  * 方法区：线程共享，存储被加载的类信息、静态变量、常量、即时编译器编译后的代码等数据。
  * 运行时常量池：方法区的一部分，class文件的常量池(字面常量和符号引用)+运行时产生的常量，其中符号引用又包含三类：类、接口的全限定名，字段名称和描述，方法名称和描述。
  * 程序计数器：  当前线程执行的字节码的行号指示器。
  * 虚拟机栈：每一个方法对应一个栈帧，栈帧 = 局部变量表、操作数栈、动态链接、方法出口信息。栈帧的大小编译时已经确定了。
  * 本地方法栈：为Native方法提供的栈。
  * 直接内存：不属于jvm管理，但是在nio中，会使用native方法申请堆外内存，并在java堆中保存其引用。

  其中，堆和方法区是所有的线程所共享的，而虚拟机栈、本地方法栈和程序计数器是各线程所独享的。

  在HotSpot虚拟机实现中，直接将虚拟机栈和本地方法栈合二为一; 而方法区则放入堆中，被称为**永久代**。并且, 方法区和堆都会参与垃圾回收, 具体见下.

* 内存分配

  内存分为新生代, 老年代和永久代.

  * 一般对象会在新生代分配内存, 少数的大对象, 如长字符串和数组, 会直接分配到老年代.
  * 新生代的对象具有存活时间短的特点, 所以Minor GC比较频繁. 每个新生代都有个年龄计数器, 每次Minor GC, 年龄加一, 一定程度后会被移入老年代

  * 老年代代表着存活时间长的特点, Major GC频率低, 但至少会伴随依次Minor GC. 

    > Minor GC和Major GC仅仅指不同代的GC而已.

  * 堆中的方法区被称为永久代, 但仍会被回收. 主要回收废弃常量和无用类, 其中判断类是否无用的条件如下:

    1. 该类所有实例都被回收
    2. 加载该类的ClassLoader被回收
    3. 对应的Class对象没有在任何地方被引用。

  

--------

其他未迁移的内容:

* [jvm学习小结一](https://blog.csdn.net/jdbdh/article/details/82495735)
* [jvm学习小结二----垃圾收集器与内存分配策略](https://blog.csdn.net/jdbdh/article/details/82529463)
* [java类型信息](https://blog.csdn.net/jdbdh/article/details/82381514)