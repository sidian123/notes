# 事务

## ACID

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

> ×表示未解决，✔表示能够解决
> mysql的默认隔离级别：repeatable_read。

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

## 锁

理解原理之前，需要知道，一般DBMS使用两端封锁协议（[Two-phase locking][4]）来实现并发控制。主要存在两种锁：写锁（独占锁）和读锁（共享锁）。一个对象可能会被加多个锁，或者只有一个锁，说明有的锁是兼容的，有的不是，如下锁兼容性表所示：

| Lock type  | read-lock | write-lock |
| ---------- | --------- | ---------- |
| read-lock  |           | X          |
| write-lock | X         | X          |

> `x`表示不兼容

一个事务尝试对已加过锁的对象加锁时，会判断两者是否兼容，如果不兼容就被阻塞，该事务加入到对象的等待队列中。

> <span style="color:red">另外</span>，不要被锁的名字给迷惑，并不是说加了写锁的对象就一定只能写入。锁只是表明一种意图，而不是规定它的操作。一般对数据增删改(DML)时会加写锁，对数据查(DQL)时会加读锁。

> 参考：
>
> * [Isolation (database systems)](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Repeatable_reads)
> * [Concurrency control](https://en.wikipedia.org/wiki/Concurrency_control)
> * [Two-phase locking](https://en.wikipedia.org/wiki/Two-phase_locking)

[4]:https://en.wikipedia.org/wiki/Two-phase_locking

# 命名

* 表名代表表中的一行

  > 如学生表, 表中每一行都代表一名学生, 表是学生的集合

* 表名应为名词, 单数.

* 前后缀

  * 同一模块中的表要有前缀
  * 除表外, 其他的用加后缀, 如
    * `_V` View (with the main `TableName` in front, of course)
    * `_fk` Foreign Key (the constraint name, not the column name)
    * `_cac` Cache
    * `_seg` Segment
    * `_tr` Transaction (stored proc or function)
    * `_fn` Function (non-transactional), etc.

> 参考[Relational table naming convention ](https://stackoverflow.com/questions/4702728/relational-table-naming-convention)

# 其他

* `union [all]`连接两个集合, 每项字段类型需一致, 无`all`时会去重