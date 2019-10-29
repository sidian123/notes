# 概述

* 进程与线程

  * *进程*是cpu资源分配的最小单位,*线程*是cpu调度的最小单位
  
  * 进程之间的上下文切换开销大, 线程开销小.  
  
  * 目前, 操作系统仅调度线程, 而非进程, 所以才说线程是CPU调度的最小单元.
  
  * 进程上下文切换仅仅指的是, 属于不同进程的线程间上下文切换的过程. 该过程仅多了一步: 重置进程虚拟地址寄存器MMU, 清空CPU缓存. 
  
    > 因此同一进程间的线程都共享资源, 所以没有上述那一步.
  
  * 很多系统都支持进程间交流(IPC), 如通过pipes和sockets; 而线程之间的沟通可通过共享资源来完成.
  
* Java中, 主线程才能创建线程.
  

# 线程

* 每个线程都与一个`Thread`实例关联

## 创建

 创建线程时必须提供要运行的代码. 有两种方式:

* 提供`Runnable`对象, 线程会执行该对象的`run()`方法

  ```java
      public static void main(String[] args) throws IOException, InterruptedException {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println("hello world");
              }
          }).start();
          System.out.println("hello    ....");
      }
  ```

* 继承`Thread`, 该类本身已经实现了`Runnable`, 只是`run()`方法未做什么.

  ```java
  public class HelloThread extends Thread {
  
      public void run() {
          System.out.println("Hello from a thread!");
      }
  
      public static void main(String args[]) {
          (new HelloThread()).start();
      }
  
  }
  ```

* 注意

  * 线程仅运行`run()`方法, 其他的是主线的资源, 但线程可以访问
  * `start()`启动了线程后不意味着线程马上执行, 因为主线程的cpu时间片未用完, 不会立马切换线程执行的.

## 操作

* 睡眠: `Thread.sleep()`暂停当前线程的执行, 释放对CPU的占有, 并持续一段时间. 

  > * 参数可传入毫秒或微妙, 但不保证精度
  >
  > * 睡眠期间可能会被其他线程中断唤醒, 并抛出`InterruptedException`异常

* 中断: 调用一个线程实例的`interrupt()`方法, 中断该线程, 并设置中断标志. 中断后会有什么效果呢?

  > 注意, 当前线程不能给自己发送中断.

  * 当线程正在执行会抛出`InterruptedException`异常的方法时, 会结束该方法的执行, 并抛出该异常, 并清理被设置的中断标志
  * 当线程未执行这类方法时, 被中断后, 会设置中断标志. 通过类方法`Thread.interrupted()`, 可判断当前线程是否被中断过, **同时该方法会清除中断标志**. 不想清除中断标志, 可使用实例方法`isInterrupted()`

  > 计算机底层所涉及到的概念, Java的中断概念与之有些相像, 但也不完全一致.

* Join: 调用线程实例的`join()`方法, 造成当前线程等待实例的线程的完成, 或者等待一定时间.

  > 同样等待时间不保证精度, 可能会被中断并抛出异常

# 并发

## 挑战

1. 资源竞争: 当一个线程对资源的操作未完成时, 其他线程来访问, 将会出错.

	> 类比数据库的事务概念, 一个事务未完成便访问, 可能会出现丢失修改,脏读, 不可重复读等现象. 因此需要一个机制, 使不同线程之间有一定的隔离性, 同步块的使用达到了数据库中serializable的隔离级别. 若想要更好的效率, 可考虑使用读写锁.

2. 可见性: 由于CPU缓存, 寄存器, 多核和优化的存在, 造成数据更新后的值没有被立马回写到内存中, 造成其他线程读取到老的值.

	> 新数据可能存在CPU缓存中, 寄存器中

3. 代码重排: 代码在优化过程中, 执行顺序会被打乱. 注意, **这本质上还是可见性的问题**, 见`volatile`

	> 这里先提一提, 对`volatile`变量的写入, 将会造成该变量及其之前的变量的缓存都被写入到内存中, 但该变量后面的写入不会被刷新. 即使源码中变量位于`volatile`变量前进行写操作, 由于代码重排, 导致不能保证`volatile`变量刷新时, 其他变量也被刷新

## synchronized

用于解决并发问题

### 介绍

每个对象一个与其关联的锁( Intrinsic Lock or Monitor Lock ), `synchronized`可定义一个同步代码块, 与一个对象的锁关联, 有如下效果:

* 第一个拿到锁的线程能够进入并执行对应的代码块, 其他想获得锁的线程将被等待

  > 解决挑战1

* 离开同步代码块时, 将缓存写入内存, 保证内存数据最新

  > 解决挑战2,3

### 使用

1. 直接在方法原型上添加`synchronized` 

   1. 如果是实例方法, 则获取实例的锁; 
   2. 如果是静态方法, 则获取类的锁, 即`Class`实例的锁.

   ```java
   public synchronized void increment() {
       c++;
   }
   ```

2. 语句块上使用, 如

   ```java
   public void addName(String name) {
       synchronized(this) {
           lastName = name;
           nameCount++;
       }
       nameList.add(name);
   }
   ```

   直接在小括号内指定获取锁的对象.

### 可重入

内部锁 ( Intrinsic Lock ) 除了具有排他性外, 还具有可重入性 ( Reentrant ) , 即已获得该锁的线程可再次获得该锁.

## 信号同步

### 介绍

同步块仅避免了线程并发带来的问题, 若要让线程之间达到一定的同步运行( 如生成者-消费者模型 ), 需要使用忙等待的机制, 十分耗费CPU. 而**信号**提供了这种解决方案.

> 甚至可通过信号, 自己实现一个锁

### 使用

简单的说, `wait()`让线程等待, 其他线程通过`notify()`等方法让线程被唤醒. 

* 等待: 使用`wait()`之前, 当前线程必须要获取到一个对象 ( 假设A ) 的锁, 之后调用`wait()`方法, 让当前线程等待并记录在对象A的等待列表 ( Wait Sets ) 中, 同时释放锁.

* 唤醒: 同样的, 若要唤醒在对象A的等待列表中的线程, 需要调用A对象的`notify()`或`notifyAll()`方法, 前提时, 当前线程同样先获取对象A的锁. 唤醒后, 执行停在`wait()`调用处, 等待重新抢占对象A的锁并执行.

  > * `notify()`仅唤醒一个线程, 且在等待列表中随机挑选的, 而`notifyAll()`唤醒等待的所有线程
  >
  > * 其他唤醒方法
  >
  >   * 使用中断, 即调用线程实例的`interrupt()`方法
  >
  >   * `wait()`设置等待时间
  >   * 自动唤醒... 是的, 莫名其妙就有被自动唤醒的可能.

> 不要将等待列表与获取锁而阻塞的列表混为一谈哦.

### 例子

```java
public class DemoApplication {


    public static void main(String[] args) throws InterruptedException {
        MyThread thread = new MyThread();
        thread.start();
        System.out.println("主线程睡三秒");
        Thread.sleep(3000);
        System.out.println("让线程起床");
        thread.up();
        thread.join();
    }


    static class MyThread extends Thread {

        @Override
        synchronized public void run() {
            try {
                System.out.println("本线程也在睡觉");
                wait();
                System.out.println("hello 世界");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        synchronized void up() {
            System.out.println("本线程要起床了");
            notify();
        }
    }
}
```

输出

```
主线程睡三秒
本线程也在睡觉
让线程起床
本线程要起床了
hello 世界
```

## 新的问题

使用了锁之后, 可能会出现下列问题:

* 死锁: 线程相互等待对方占有的资源, 而一直被阻塞

* 饿死: 由于自身优先级低, 一直被后来的线程抢占资源, 从而饿死

* 活锁: 线程间必须达成某种为真的条件才能执行下去, 但是一直处于检查到为假的结果, 即使线程间状态在改变.

  > 这种线程不会被堵塞, 最耗费CPU

# High API

上述讲的都是较低层的并发API的使用, 这里介绍JDK提供的更高级的API的使用.

## Lock

未完待续...

# 其他

  不可变的对象, 一旦被构建, 它的使用便是线程安全的. 

# 参考

* [oracle tutorial](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)

* [java.util.concurrent](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html)