[TOC]

# 一 介绍
JDBC（Java Database Connectivity）是Java程序访问数据源（主要指数据库）的统一编程接口。它屏蔽了所有数据库访问的细节，使数据库操作更为简单。

JDBC用接口Connection表示应用与数据库的连接;通过Connection可以获得Statement，它表示sql语句；执行Statement后，可能会得到ResultSet，它表示类似表格结构的结果集；Cursor指向结果集的某一行，当取出结果集数据时，需要通过Cursor遍历所有行，然后一行一行地将结果集取出。

>JDBC不仅仅可以访问数据库，还可以访问文件系统和一些其他的数据源，这些统称为**数据源**（data source），但要实现数据源对应JDBC驱动。
# 二 使用
使用JDBC访问数据库就是一个获取连接（Connection）、执行sql语句（Statement）、获取结果集（ResultSet、Cursor）和关闭资源的过程。

--------

在获取Connection之前，需要加载**JDBC 驱动**，因为为了屏蔽所有数据库访问的细节，oracle只定义了接口，需要数据库供应商提供实现，即**JDBC driver**。但自从JDBC4.0以来，DriverManager会自动从classpath上加载存在的驱动，低版本的JDBC则需要手动加载（通过Class.forName方法）。

------

有两种方法可以获得Connection：
* **DriverManager**：代表驱动管理器，获取连接最原始、最简单的方式。
	
	>查阅源码，会发现它的静态初始化块会自动在classpath上加载驱动（需4.0及更高版本）。
* **DataSource**：表示数据源，一个产生连接的工厂，它是更高级的抽象，因此有更多的功能。它是一个接口，实现分三类：
	* 产生于DriverManager一致的Connection。
	* 与pooling管理器合作，产生与连接池相关的Connection。
	* 与事物管理器合作，产生与分布式事务相关的Connection。
	
	>含有分布式事务功能的数据源一般也会实现连接池。

-----------

Connection表示应用与数据源的连接，也是sql语句执行、事务处理的上下文（context），有一些默认的环境配置，如默认自动提交（autocommit为true）、默认的事务隔离级别。
* 默认配置随着数据库的不同而不同，Connection提供了方法查询这些设置和数据库的提供的功能。
* 还提供了事务处理的功能，如回滚、提交、设置savepoint。
* 也许Connection并不代表真正的物理连接，而是它的句柄（handler），如与连接池相关的Connection，关闭它只会让它回收到连接池中，而不是真正的被关闭。

-----------

Statement表示sql语句，分三类：Statement、PreparedStatement和CallableStatement。ResultSet可以配置三种特性：type、concurrency和cursor holdability。

然后就是执行语句，通过Cursor获取结果集。之后要注意关闭资源。没什么好说的，见下文。


# 三 JDBC驱动
JDBC主要定义了接口，需要数据库供应商提供实现，即**JDBC driver**。

JDBC驱动可分为四类：
1. 驱动将JDBC API接口**映射**到其他数据访问API上，如JDBC-ODBC桥。
2. 驱动**主要**由java语言实现，部分数据相关的功能由本地代码实现。
3. 驱动作为**纯java客户端**，与**中间层服务器**交互，服务器使用数据库独立的协议，处理驱动的数据访问请求。
4. 驱动使用**纯java**编写，实现**网络协议**，直接连接到数据源。

而MySQL的驱动实现（[Connector/J][1]）属于第四种。

>ODBC也是一种访问数据库的API接口，但不局限于Java语言。数据库供应商同样也会提供ODBC驱动，但是不一定提供JDBC驱动，因此Java提供了JDBC-ODBC桥来访问这些数据库，即JDBC驱动通过ODBC来实现。

[1]:https://www.mysql.com/products/connector/

# 四 DataSource
之前提到，获得Connection有两种方法，而DriverManager是最简单的一种方法，它获得标准的Connection。JDBC的操作过程一般是，获得Connection，执行sql操作，关闭Connection。每次sql操作都要打开和关闭Connection，开销较大。如果使用连接池，每次关闭Connection时，让连接返回到连接池，而不是真正关闭；获得连接时，从连接池中获得可用连接，这样性能会得到极大提高。但标准的Connection不能完成这个功能。如果要在连接上处理分布式事务时，标准的Connection也不能胜任。

因此出现了DataSource接口，它是更高级的抽象，表示数据源，用于产生Connection。通过不同的实现，可以产生的不同的Connection，完成不同的任务。

-----------
在oracle的[教程][2]中，DataSource的实现中，需要先实现接口ConnectionPoolDataSource或XADataSource，它们产生PooledConnection或XAConnection，这些连接代表真正的物理链接。然后实现对应的DataSource接口，该接口才是开发人员使用的接口，通过该接口获得Connection。然而Connection却不是真正的物理连接，只是PooledConnection或XAConnection的句柄（handler），Connection的功能都会通过PooledConnection或XAConnection实现。
>实现了分布式事务的DataSource一般也会实现连接池
>PooledConnection和XAConnection应该是数据库供应商提供的吧

DataSource实现不唯一，比如含有连接池的DataSource，先继承Connection，实现新Connection，重写它的close()方法，让连接关闭时被回收到连接池中；而DataSource中实现连接池管理功能，并生产新Connection。
>Connection应该是DriverManager提供的吧。

------------
目前的DataSource一般将连接池管理器内置到DataSource中，而事务管理器分离出来。**总之**，DataSource的目的是为了产生含有不同功能的Connection，如果Connection是标准的Connection，那么它和DriverManager就没啥子区别了。因此我在使用DataSource时，和往常使用一般无二。

>注意，此章含有大量个人猜想，可能不对。

[2]:https://docs.oracle.com/javase/tutorial/jdbc/basics/sqldatasources.html

# 五 Statement
通过Connection创建Statement，代表sql语句，共三种：
* **Statement**: Used to implement simple SQL statements with **no parameters.**
* **PreparedStatement**: (Extends Statement.) Used for precompiling SQL statements that **might contain input parameters**.
* **CallableStatement**: (Extends PreparedStatement.) Used to execute stored procedures that **may contain both input and output parameters**. 

Statement在执行SQL语句时才需要给出sql语句，然后给DBMS编译执行。而PrepareStatement在被创建时就给出SQL（含占位符），然后编译；当需要执行时，才给出剩下的参数，后让DBMS直接执行。因此一条sql语句多次执行，只是参数不同时，使用PrepareStatement效率高。至于CallableStatement，用于执行存储过程，用的不多，不提也罢。

# 六 Connection
通过`DriverManager`或`DataSource`可以将应用链接到数据源上，即获得连接（`Connection`）。

* **DriverManager**：通过指定数据库URL，连接应用到数据源上。对于不低于**4.0**版本的JDBC，当该类被加载时，会**自动**加载classpath上的JDBC驱动。
* **DataSource**：该接口是一个更好的选择，因为它屏蔽了底层数据源的细节。（*我猜想，该接口的实现类就是通过DriverManager实现的，然后配置好连接，用户直接从DataSource中取出Connenction使用。*）

在jdbc4.0之前，连接数据库需要通过`Class.forName`方法手动加载JDBC驱动，这些驱动实现了`java.sql.Driver`。

更多内容见第四章。
# 七 ResultSet和Cursor
ResultSet表示查询后的结果集，通过Cursor可以取出数据。通过ResultSet也可以修改数据库数据。
## ResultSet
ResultSet有三方面可以设置：type、concurrency和cursor holdability，在创建Statement时设置。
* type：设置Cursor如何滚动，可前滚还是后滚，底层数据变动是否会反应到ResultSet上。默认TYPE_FORWARD_ONLY。
* concurrency：设置是否ResultSet能够更新数据库数据，默认CONCUR_READ_ONLY，即不能。
* Cursor Holdability：控制commit时，ResultSet是否被关闭。默认设置取决于DBMS，但是Connection默认查询完后就自动提交的，因此一般默认HOLD_CURSORS_OVER_COMMIT吧？这样才能在执行sql后正常取出数据。

---------------------
ResultSet提供了一系列获得结果集中列的方法，这里给出了MySQL中数据类型和JDBC类型的对应关系：[Java, JDBC and MySQL Types][3]

[3]:https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-type-conversions.html

## Cursor
默认Cursor（光标）位于第一行前面，一般通过next方法移动Cursor，指向下一行。还提供更多的方法来移动Cursor，有些方法需要ResultSet设置为允许前滚才能使用。

# 八 事务
事务（Transaction）是一组SQL语句，被视作一个工作单元，所有的语句要么全部执行，要么全部不被执行。事务必须满足ACID：
* **Atomicity**（原子性） − A transaction should be treated as a single unit of operation, which means either the entire sequence of operations is successful or unsuccessful.
* **Consistency**（一致性） − This represents the consistency of the referential integrity of the database, unique primary keys in tables, etc.
* **Isolation**（隔离性） − There may be many transaction processing with the same data set at the same time. Each transaction should be isolated from others to prevent data corruption.
* **Durability**（持久性） − Once a transaction has completed, the results of this transaction have to be made permanent and cannot be erased from the database due to system failure.

## 隔离性
隔离性主要为了让事务之间互不干扰，但良好的隔离性和性能不可兼得，因此出现了不同的隔离级别，解决不同的问题：
| Isolation Level              | 丢失修改 | Dirty Reads | Non-Repeatable Reads | Phantom Reads |
| :--------------------------- | :------- | :---------- | :------------------- | :------------ |
| TRANSACTION_NONE             | ×        | ×           | ×                    | ×             |
| TRANSACTION_READ_UNCOMMITTED | ✔        | ×           | ×                    | ×             |
| TRANSACTION_READ_COMMITTED   | ✔        | ✔           | ×                    | ×             |
| TRANSACTION_REPEATABLE_READ  | ✔        | ✔           | ✔                    | ×             |
| TRANSACTION_SERIALIZABLE     | ✔        | ✔           | ✔                    | ✔             |
×表示未解决，✔表示能够解决
>mysql的默认隔离级别：repeatable_read。



和隔离性相关的问题如下：
* 丢失修改：事务A读取值，之后事务B也读取值并修改然后提交，事务A修改后提交。此时事务B修改的值被丢失。
* 脏读：事务A读取事务B尚未提交的值。如果事务B回滚，则事务A读的值就“脏”了。
* 不可重复读：事务A读取值，事务B修改该值并提交，事务A再次读取该值，发现值不同了。
* 幻读：事务A根据一定条件读取一组数据，事务B插入一组满足该条件的数据并提交，事务A再根据同样条件读取一组数据，发现多了一组数据。

隔离性的实现和锁有关，下面通过讲述各个隔离级别的实现原理：
* read_uncommited：加写锁，直到事务结束释放，解决丢失修改问题。
* read_commited:在上一级别基础上加读锁（可能为乐观锁），读完立即释放，解决脏读。
* repeatable_read：在read_uncommited基础上加读锁（可能为乐观锁），直到事务结束释放，解决不可重复读问题。
* serializable：在上一级别基础上加间隙锁等等等等，因此解决了幻读。

--------------
理解原理之前，需要知道，一般DBMS使用两端封锁协议（[Two-phase locking][4]）来实现并发控制。主要存在两种锁：写锁（独占锁）和读锁（共享锁）。一个对象可能会被加多个锁，或者只有一个锁，说明有的锁是兼容的，有的不是，如下锁兼容性表所示：
| Lock type  | read-lock | write-lock |
| ---------- | --------- | ---------- |
| read-lock  |           | X          |
| write-lock | X         | X          |
x表示不兼容

一个事务尝试对已加过锁的对象加锁时，会判断两者是否兼容，如果不兼容就被阻塞，该事务加入到对象的等待队列中。

另外，不要被锁的名字给迷惑，并不是说加了写锁的对象就一定只能写入。锁只是表明一种意图，而不是规定它的操作。一般对数据增删改(DML)时会加写锁，对数据查(DQL)时会加读锁。

参考：
[Isolation (database systems)](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Repeatable_reads)
[Concurrency control](https://en.wikipedia.org/wiki/Concurrency_control)
[Two-phase locking](https://en.wikipedia.org/wiki/Two-phase_locking)

[4]:https://en.wikipedia.org/wiki/Two-phase_locking


## 使用
Connection提供了commit、rollback、savepoint功能。一个连接对应一个事务吧？ 使用事务之前，必须设置autocommit为false，因为默认事务自动提交的。commit提交事务，rollback回滚事务到开始处，或者提供参数savepoint，回滚到保存点。

显式执行commit方法或执行DDL语句（隐式提交）时会提交事务。提交事务意味着旧事务的结束和新事务的开始。

Connection开启一个会话，默认自动提交，但是在涉及事务处理时，连接默认不自动提交（针对于框架，如mybatis）。

# 九 其他
## URL
在连接时，需要通过URL来连接数据库。URL可以指定数据库位置，给出账户密码，配置连接信息等。连接数据库的URL在每个DBMS中都会不同。

下面给出连接MySQL的URL，语法如下：
```syntax
jdbc:mysql://[host][,failoverhost...]
    [:port]/[database]
    [?propertyName1][=propertyValue1]
    [&propertyName2][=propertyValue2]...
```
* **host:port** is the host name and port number of the computer hosting your database. If not specified, the default values of host and port are 127.0.0.1 and 3306, respectively.
* **database** is the name of the database to connect to. If not specified, a connection is made with no default database.
* **failover** is the name of a standby database (MySQL Connector/J supports failover).
* **propertyName=propertyValue** represents an optional, ampersand-separated list of properties. These attributes enable you to instruct MySQL Connector/J to perform various tasks.

>在url中也可以不指定数据库名，可以通过执行sql语句来执行，如`use databaseName`。

## 关闭资源
打开的资源不用时，需要关闭，如Connection、Statement、ResultSet都需要关闭。

但**一般我们只需要关闭Connection和Statement**，因为当Statement被关闭、重新执行或从多个结果序列中取出下一个时，都会自动关闭ResultSet。

关闭Connection时，会关闭其他资源（含statement和ResultSet）。但是不建议这么做，因为在连接池中并不会真正的关闭连接，因此建议手动关闭statement和ResultSet。

## javax.sql和java.sql
java.sql主要提供了java客户端所需要的JDBC功能，而javax.sql扩展了java.sql，提供了企业版的功能。但我觉得，javax.sql中除了DataSource接口，其他的也不怎么常用。

# 参考
https://docs.oracle.com/javase/tutorial/jdbc/TOC.html