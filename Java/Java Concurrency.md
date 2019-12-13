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

2. 可见性(或一致性): 由于CPU缓存, 寄存器, 多核和优化的存在, 造成数据更新后的值没有被立马回写到内存中, 造成其他线程读取到老的值.

	> 新数据可能存在CPU缓存中, 寄存器中

3. 代码重排: 代码在优化过程中, 执行顺序会被打乱. 注意, **这本质上还是可见性的问题**, 见`volatile`

	> 这里先提一提, 对`volatile`变量的写入, 将会造成该变量及其之前的变量的缓存都被写入到内存中, 但该变量后面的写入不会被刷新. 即使源码中变量位于`volatile`变量前进行写操作, 由于代码重排, 导致不能保证`volatile`变量刷新时, 其他变量也被刷新

> 外文中经常提到***Happens before relationship*** , 指的是一个保证, 即一个线程的操作对其他线程是可见的. 也就是, 保证*happen before*关系, 就是保证上述2,3点条件. *happen before* 本质就是**一致性**问题, 见[Memory Consistency Properties](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/package-summary.html#MemoryVisibility).

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

## 集合

### BlockingQueue

#### 介绍

![A BlockingQueue with one thread putting into it, and another thread taking from it.](.Java%20Concurrency/blocking-queue.png)

* 代表队列的接口, 能够线程安全的添加和删除元素. 
* 能够阻塞线程, 如试图向空队列中取出元素时, 将阻塞, 直到队列中新增元素; 同样的, 试图向满队列添加元素也将被阻塞.

#### 操作

增, 删, 检查三种操作有四种不同类型: 

|             | **Throws Exception** | **Special Value** | **Blocks** | **Times Out**                 |
| ----------- | -------------------- | ----------------- | ---------- | ----------------------------- |
| **Insert**  | `add(o)`             | `offer(o)`        | `put(o)`   | `offer(o, timeout, timeunit)` |
| **Remove**  | `remove(o)`          | `poll()`          | `take()`   | `poll(timeout, timeunit)`     |
| **Examine** | `element()`          | `peek()`          |            |                               |

如果操作暂时不能进行时, 分别将 

1. 抛出异常 
2. 返回特殊值(`true` or `false` 
3. 阻塞
4. 超时而返回特殊值(`true` or `false`)

> 其他注意点
>
> 1. 不能插入`null`
> 2. 可以删除中间元素, 但耗时

#### 实现

> 有界指队列容量有限, 无界指队列元素可以无限增加

* `ArrayBlockingQueue`

  由数组实现的有界的阻塞队列. 一旦被创建, 容器大小将被固定.

* `DelayQueue`

  无界的阻塞队列,  元素有过期时间. 获取元素时, 线程将被阻塞, 直到一个元素过期. 元素需实现`Delayed`接口, 来提供过期时间.

* `LinkedBlockingQueue`

  由链表实现的, *optionally-bounded* 的阻塞队列. 创建实例时, 若未指定容量大小时, 将默认`Integer.MAX_VALUE`, 即无界; 若指定容量大小, 将有界.

* `PriorityBlockingQueue`

  无界的阻塞队列, 元素有优先级, 通过`Comparable`或`Comparator`给出. 元素越小, 优先级越高.

* `SynchronousQueue`

  容器容量为0的阻塞队列, 即插入数据将立刻等待, 直到消费者取出.

### BlockingDeque

`BlockingDeque`是一个阻塞双端队列, 继承于`BlockingQueue`. 

![A BlockingDeque - threads can put and take from both ends of the deque.](.Java%20Concurrency/blocking-deque.png)

同样有四类方法, 但有两组 

|             | **Throws Exception** | **Special Value** | **Blocks**     | **Times Out**                      |
| ----------- | -------------------- | ----------------- | -------------- | ---------------------------------- |
| **Insert**  | `addFirst(o)`        | `offerFirst(o)`   | `putFirst(o)`  | `offerFirst(o, timeout, timeunit)` |
| **Remove**  | `removeFirst(o)`     | `pollFirst(o)`    | `takeFirst(o)` | `pollFirst(timeout, timeunit)`     |
| **Examine** | `getFirst(o)`        | `peekFirst(o)`    |                |                                    |

|             | **Throws Exception** | **Special Value** | **Blocks**    | **Times Out**                     |
| ----------- | -------------------- | ----------------- | ------------- | --------------------------------- |
| **Insert**  | `addLast(o)`         | `offerLast(o)`    | `putLast(o)`  | `offerLast(o, timeout, timeunit)` |
| **Remove**  | `removeLast(o)`      | `pollLast(o)`     | `takeLast(o)` | `pollLast(timeout, timeunit)`     |
| **Examine** | `getLast(o)`         | `peekLast(o)`     |               |                                   |

> 详细见上一小节

其实现类为`LinkedBlockingDeque`, 与`LinkedBlockingQueue`类似, 见上一小节.

### 其他集合

* Map
  * `ConcurrentHashMap`对应`HashMap`
  * `ConcurrentSkipListMap`对应`TreeMap`
  * ...

* List

  * `CopyOnWriteArrayList`对应 `ArrayList` 

  > 没了, `LinkedList`无对应的并发实现

* Set 

  * `ConcurrentSkipListSet`对应`TreeSet`
  *  `CopyOnWriteArraySet`无对应, 由`CopyOnWriteArrayList`实现的

## 同步方案

### CountDownLatch

类似于门闩, 需要一定条件才能打开, 未打开时, 所有达到的线程需等待. 

使用方式如下

1. `CountDownLatch`以一个数值初始化
2. 调用`await()`方法的线程将被阻塞, 直到其他线程调用`countDown()`将数值减一至零为止, 才恢复执行.

> `CountDownLatch`不可重置数值, 即不可重复使用

例子: 线程们同步执行, 主线程等到所有线程结束才结束

```java
 class Driver { // ...
   void main() throws InterruptedException {
     CountDownLatch startSignal = new CountDownLatch(1);
     CountDownLatch doneSignal = new CountDownLatch(N);

     for (int i = 0; i < N; ++i) // create and start threads
       new Thread(new Worker(startSignal, doneSignal)).start();

     doSomethingElse();            // don't let run yet
     startSignal.countDown();      // let all threads proceed
     doSomethingElse();
     doneSignal.await();           // wait for all to finish
   }
 }

 class Worker implements Runnable {
   private final CountDownLatch startSignal;
   private final CountDownLatch doneSignal;
   Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
     this.startSignal = startSignal;
     this.doneSignal = doneSignal;
   }
   public void run() {
     try {
       startSignal.await();
       doWork();
       doneSignal.countDown();
     } catch (InterruptedException ex) {} // return;
   }

   void doWork() { ... }
 }
```

### CyclicBarrier

类似一个障碍点, 需要固定数目线程的到来, 才能突破障碍点, 此时所有线程可通过. 还支持突破障碍点前执行可选的任务; 和重置障碍点.

使用很简单, 调用`await()`造成当前线程等待该障碍点被突破, 有多种方法可突破: 

1. 固定数量的线程的到来
2. 当前线程被中断, 将抛出`InterruptedException`
3. 该障碍点中任意一个线程被中断, 将抛出`BrokenBarrierException`
4. 该障碍点中任意一个线程超时, 将抛出`BrokenBarrierException`

5. 该障碍点被重置, 将抛出`BrokenBarrierException`
6. 其他原因, 见`BrokenBarrierException`

例子: 两个障碍点, 两个线程

```java
public class CyclicBarrierRunnable implements Runnable{

    CyclicBarrier barrier1 = null;
    CyclicBarrier barrier2 = null;

    public CyclicBarrierRunnable(
            CyclicBarrier barrier1,
            CyclicBarrier barrier2) {

        this.barrier1 = barrier1;
        this.barrier2 = barrier2;
    }

    public void run() {
        try {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() +
                                " waiting at barrier 1");
            this.barrier1.await();

            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() +
                                " waiting at barrier 2");
            this.barrier2.await();

            System.out.println(Thread.currentThread().getName() +
                                " done!");

        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}

public class Main{
    public static void main(String[] args){
        Runnable barrier1Action = new Runnable() {
            public void run() {
                System.out.println("BarrierAction 1 executed ");
            }
        };
        Runnable barrier2Action = new Runnable() {
            public void run() {
                System.out.println("BarrierAction 2 executed ");
            }
        };

        CyclicBarrier barrier1 = new CyclicBarrier(2, barrier1Action);
        CyclicBarrier barrier2 = new CyclicBarrier(2, barrier2Action);

        CyclicBarrierRunnable barrierRunnable1 =
                new CyclicBarrierRunnable(barrier1, barrier2);

        CyclicBarrierRunnable barrierRunnable2 =
                new CyclicBarrierRunnable(barrier1, barrier2);

        Thread thread1 = new Thread(barrierRunnable1);
        Thread thread2 = new Thread(barrierRunnable2);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
    }
}
```

结果

```output
Thread-0 waiting at barrier 1
Thread-1 waiting at barrier 1
BarrierAction 1 executed
Thread-1 waiting at barrier 2
Thread-0 waiting at barrier 2
BarrierAction 2 executed
Thread-0 done!
Thread-1 done!
```

### Exchanger

一个同步点, 先到达的两个线程可以交换对象.

使用: 调用`exchange()`, 传入交换的对象, 等待另一个线程的到来, 即调用`exchange()`

例子: 一个生产者填充好数据后, 交换消费者榨干的数据.

```java
class FillAndEmpty {
   Exchanger<DataBuffer> exchanger = new Exchanger<>();
   DataBuffer initialEmptyBuffer = ... a made-up type
   DataBuffer initialFullBuffer = ...

   class FillingLoop implements Runnable {
     public void run() {
       DataBuffer currentBuffer = initialEmptyBuffer;
       try {
         while (currentBuffer != null) {
           addToBuffer(currentBuffer);
           if (currentBuffer.isFull())
             currentBuffer = exchanger.exchange(currentBuffer);
         }
       } catch (InterruptedException ex) { ... handle ... }
     }
   }

   class EmptyingLoop implements Runnable {
     public void run() {
       DataBuffer currentBuffer = initialFullBuffer;
       try {
         while (currentBuffer != null) {
           takeFromBuffer(currentBuffer);
           if (currentBuffer.isEmpty())
             currentBuffer = exchanger.exchange(currentBuffer);
         }
       } catch (InterruptedException ex) { ... handle ...}
     }
   }

   void start() {
     new Thread(new FillingLoop()).start();
     new Thread(new EmptyingLoop()).start();
   }
 }
```

### Semaphore

> 信号量, 大学操作系统的课本上就是以这个为例子的

信号量维护一组**许可**, 线程调用`acquire()`将获取信号量中的许可, 如果无可用许可将阻塞线程. 调用`release()`将添加一个许可, 被阻塞的线程中的一个将获取该许可并恢复执行.

> * 许可数目在构造是指定, 但仍可变的
> * 没有特别规定必须获取许可的线程来调用`release()`

信号量主要用于两种场景

1. 限制访问资源的线程的数量, 一般此时许可数量大于1

2. 用作互斥锁

   > 与其他锁相比, 信号量构成的锁不必须让获取锁的线程释放. 该特性可用于死锁的恢复.

等待许可的线程中, 哪个线程被恢复是不确定的, 可能会造成线程饿死的情况. 因此构造函数提供了`fair`参数, 可指定线程恢复的过程是公平的, 即先入先出FIFO.

例子: 限制访问池中元素的线程的个数

```java
 class Pool {
   private static final int MAX_AVAILABLE = 100;
   private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);

   public Object getItem() throws InterruptedException {
     available.acquire();
     return getNextAvailableItem();
   }

   public void putItem(Object x) {
     if (markAsUnused(x))
       available.release();
   }

   // Not a particularly efficient data structure; just for demo

   protected Object[] items = ... whatever kinds of items being managed
   protected boolean[] used = new boolean[MAX_AVAILABLE];

   protected synchronized Object getNextAvailableItem() {
     for (int i = 0; i < MAX_AVAILABLE; ++i) {
       if (!used[i]) {
         used[i] = true;
         return items[i];
       }
     }
     return null; // not reached
   }

   protected synchronized boolean markAsUnused(Object item) {
     for (int i = 0; i < MAX_AVAILABLE; ++i) {
       if (item == items[i]) {
         if (used[i]) {
           used[i] = false;
           return true;
         } else
           return false;
       }
     }
     return false;
   }
 }
```

> 注意, `acquire()`不会影响同步块加锁.

### Phaser

略, 待完善

### Lock

## Executor

执行任务`Runnable`的接口, 提供一种将**任务如何运行**与**任务提交**, **线程使用细节**, **调度** 解耦开来的方案. 

任务可能在新线程中执行, 可能在已存在的线程中执行, 可能在调用者线程中执行, 可能异步执行, 可能同步执行, 取决于具体使用的实现类.

### ExecutorService

* 介绍

  一个用于执行异步任务的接口, 继承于`Executor`, 并提供了额外的功能:

  * 终止任务
  * 产生`Future`对象, 跟踪异步任务的进程

* 关闭

  此时, 将拒绝新任务的加入

  * `shutdown() `执行完等待的任务才结束
  * `shutdownNow()` 不执行等待的任务, 并尽最大力去停止当前执行的任务.

* 提交任务

  * `submit()`

    与`execute()`方法相比, `submit()`方法将返回`Future`对象, 通过该对象可以取消任务或等待其执行完毕.

  * 执行集合

    * `invokeAll()`

      执行集合中所有任务并等待, 任务都完成时返回`Future`的集合.

      > 一个完成的任务指正常结束或抛出异常.

    * `invokeAny()`

      执行集合中所有任务并等待, 直到一个任务正常结束(即不抛出异常), 之后取消所有的任务. 

      若有任务抛出异常, 同样也取消所有任务, 并抛出异常`ExecutionException`

* `Future`

  代表异步操作的结果

  可以获取任务执行的结果, 取消任务, 检查任务状态

* `Executors`

  含构建`ExecutorService`实例的工具类.

### ThreadPoolExecutor

#### 介绍

* 使用线程池的执行器, 实现了`ExecutorService`接口

* 目的
  * 使用线程池, 减少任务调用的性能开销;
  * 资源的管理, 如约束线程数量
* 用法与`ExecutorService`一致.

#### 原理

`ThreadPoolExecutor`中存在一个**队列**和**线程池**. 线程池的数量由` corePoolSize`和` maximumPoolSize`约束. 

当提交任务时, 他们的交互关系如下:

* 如果 线程池数目 < `corePoolSize`, 则新增线程完成任务, 即使线程池中有空闲的线程.
* 如果 ` maximumPoolSize` > 线程池数目 >= `corePoolSize`, 且队列未满, 则任务入队, 否则新增线程完成任务
* 如果 线程池数目=` maximumPoolSize` , 则**拒绝策略**处理任务.

#### 拒绝策略

* `ThreadPoolExecutor.AbortPolicy`(默认)

  一旦被拒绝, 将抛出运行时异常`RejectedExecutionException` 

* `ThreadPoolExecutor.CallerRunsPolicy`

  一旦被决绝, 在调用`execute()`的线程中自己执行任务

* `ThreadPoolExecutor.DiscardPolicy`

  一旦被拒绝, 简单的忽略该任务

* `ThreadPoolExecutor.DiscardOldestPolicy`

  丢弃队列头上的任务(未被执行), 重新尝试执行该任务.

#### 队列使用

队列的使用大致分为三类

* 0容量的队列, 如`SynchronousQueue`

  > 任务到来, 将被直接交给空闲线程或新线程处理

* 无界的队列, 如`LinkedBlockingQueue`

  > 超出`corePoolSize`的任务将被缓存到队列中, ` maximumPoolSize`将失效

* 有界的队列, 如`ArrayBlockingQueue`

  > 队列容量大, ` maximumPoolSize`小时, CPU使用率, 系统资源和上下文切换的开销较小; 队列容量小, ` maximumPoolSize`大时, 开销较大.

> 无界(unbounded)或无限在数量上指`Long.MAX_VALUE`

#### 其他

* `keepAliveTime`

  默认指超出`corePoolSize`的线程在空闲状态下的存活时间.

  设置`allowCoreThreadTimeOut(boolean)`为`true`时可将该策略同时作用到` corePoolSize`内的线程上.

* 预启动线程

  默认首次新任务到来时才启动线程. 若构造时使用的队列不为空, 可通过`prestartCoreThread()`或 `prestartAllCoreThreads()`预启动线程来处理.

* 提供了回调函数, 略

* ... 见[ThreadPoolExecutor](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)

### ScheduledThreadPoolExecutor

* 介绍

  * 继承`ThreadPoolExecutor`, 实现了`ScheduledExecutorService`, 拥有延迟执行和周期性执行的功能.

  * 使用无界的队列, 和默认的拒绝策略.

* 使用

  * 延迟执行`schedule()`

  * 周期执行

    `scheduleAtFixedRate()` 以两个任务的开始时间区间作为周期执行

    `scheduleWithFixedDelay()` 以上一个任务的结束与下一个任务的开始为周期执行.

## Atomic

## 其他

### ThreadLocal

- 介绍

  `ThreadLocal`类型的字段是线程局部的, 意味着即使是类变量, 在不同的线程中也有不同的拷贝.

- 使用

  `ThreadLocal`字段通常声明为`private static`, 然后重写它的`initialValue`方法来设置它的初始值. 而`get`, `set`方法用于设置或获得该字段的值.

- 原理

  当线程首次调用`get`方法时, 会执行它的`initialValue`方法来初始化, 并返回该值. 之后的`get`调用仅仅只是获取该值.

  当线程die并且无其他引用时, 该变量go die, 并交由垃圾收集器处理.

> 参考: [ThreadLocal<T>](https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html)

### Timer



# 其他

## 不可变对象

  不可变的对象, 一旦被构建, 它的使用便是线程安全的. 



# 参考

* [oracle tutorial](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)
* [java.util.concurrent](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html)
* [Concurrent Jenkov.com](http://tutorials.jenkov.com/java-util-concurrent/index.html) 内容覆盖面广, 但十分简略, 建议搭配Javadoc学习