---
title: Java并发编程
tags: 
  - IO
  - 锁
  - JDK
categories: 学习笔记
top: 0
copyright: ture
date: 2019-08-13 00:20:10
---
并发编程的本质（微观:可见性、原子性、有序性；宏观:分工、协作、互斥）

AIO、BIO、NIO

同步（synchronized）、互斥（锁）

线程池

<!--more-->

# 先从IO说起

BIO（blocking IO）：阻塞IO，用户线程进行IO操作时，有可能造成线程的阻塞会知道内核将数据准备好之后，再解除阻塞状态，回到待执行队列中，进行IO操作。

NIO（no-blocking IO）：非阻塞IO。

- 典型的非阻塞IO，是采用一个死循环的机制，进行对内核的IO请求，直到内核将数据准备好，则跳出循环进行IO的读写。

- 多路复用IO，采用一个selector内核线程，去轮询用户线程，是否需要进行IO操作，需要则替用户线程去准备数据，然后也是由用户现场进行IO读写。

  缺点就是：当某一个用户线程的IO块较大时，会影响后续的用户线程以及新的IO事件。

- 信号驱动IO，用户线程在需要IO操作时，会向内核注册一个IO操作事件，然后内核去准备数据，数据在缓存区准备好之后，会再次通知用户线程来进行数据读取。

AIO（asyn IO)：异步驱动IO，用户线程需要IO操作时，会通知内核，然后内核会直接将数据拷贝到用户线程中，用户线程无需等待数据核实操作完成，即可继续执行下面的操作。



NIO适用于连接数多且连接较短的场景比如聊天服务器。AIO适用于连接数多且连接较长的场景比如相册上传服务器。实际上AIO与NIO相比，性能并没有显著提升，因此Netty采用的是异步时间驱动的NIO模型。

---

# synchronized

当修饰静态方法的时候，锁定的是当前类的Class对象，
当修饰非静态方法的时候，锁定的是当前实例对象this

synchronized关键字 及wait()、notify()、notifyAll()共同组成了 [管程](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/BlockedQueue.java)（管理共享变量 以及对共享变量的操作过程） 

与Lock相比：
 - 都是可重入锁
 - synchronized是非公平锁。ReentrantLock默认是分非公平锁，可手动设为公平锁。（ new ReentrantLock(true); ）
 - synchronized不可中断（只有在执行过程中抛异常、或执行结束，才会自动释放锁）  
    LockInterruptibly()方法可以中断自己，也可以被其他线程中断。Lock必须手动unlock()，否则会造成死锁
    - 使用synchronized，有时等待线程会一直等下去，不会响应中断
    - Lock 还可以用await() 方法让出锁资源，同时调用notify() 通知其他线程来重新获取资源。

synchronized适合低并发的场景，锁竞争发生的概率很小，此时锁处于偏向锁或者轻量级锁状态，因此性能更加好，比如jdk1.8concurrentHashMap使用synchronized的原因就是，每个hash槽上的锁竞争很少，用synchronized比lock更好

# 锁

## AQS

AbstractQueuedSynchronizer 是一个抽象类，内部提供了一个基于CLH 自旋锁实现的一个隐式的FIFO队列，来管理多线程间的同步状态。

独占锁（ReentrantLock），内部实现的是AQS的acquire() 和release()方法。  
共享锁（ReeantrantReadWriteLock, Semaphore, CountDownLatch），内部实现的是AQS的acquireShared() 和releaseShared() 方法。

以ReentrantLock 的公平锁和非公平锁为例，详解获取和释放锁的过程 使用CAS的方式去尝试修改state变量。   
[AQS](https://github.com/nibnait/algorithms/tree/master/src/main/java/jdk/concurrent/AQS)

## 信号量

使用Semaphore 信号量来对对象池进行限流 [ObjectPool](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/ObjectPool.java)

## 读写锁

ReadWriteLock 读锁无法升级为写锁。但是支持锁的降级 [Cache](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/Cache.java)

StamptedLock 是不可重入锁。支持锁的降级（通过tryConvertToReadLock() 方法实现）和升级（通过tryConvertToWriteLock() 方法实现）[Point](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/Point.java)

## 如何预防死锁

造成死锁的条件：

1. 共享资源本身互斥（只能被1个线程占用）
2. “占用且等待”：A线程已取得共享资源X，在等待资源Y时，不会释放X
3. 线程内部资源“不可抢占“
4. “循环等待”：A等待B的Y，B等待A的X

解决方案：

- 破坏“占用且等待”条件：创建线程前，一次性申请所有资源。
- 破坏“不可抢占”条件：设置超时机制，如果A线程迟迟获取不到某资源，超时主动释放它所占有的全部资源
- 破坏“循环等待”条件：将资源编号，X(1号)，Y(2号)，线程申请资源时，按编号顺序申请，在拿到Y之前必须先申请X。

### 最佳实践

- 永远只在更新对象时加锁
- 永远只在访问可变成员时加锁
- 永远不在调用其他对象的方法时加锁（防止“其他”方法有线程的sleep()、I/O操作等影响性能）

### 无锁化编程

1. COW（CopyOnWriteArrayList）

   读的是原array。写操作时是将原array复制出来，写操作完毕，再将原array执行新array[0]的地址。因此迭代时对原array进行修改是无效的。

   只适用于读多写少 且允许读写短暂不一致的场景。

2. CAS（CompareAndSwap）——ABA问题：

   线程1 expect=100元, update=50元
   线程2 expect=100元, update=50元
   假设银行的提款机采用普通的CAS方式来进行更新余额操作。小明打算取50块钱，第一次点击提交产生线程`1`，没反应（线程`1`阻塞了），第二点击提交，产生线程`2`，线程`2`取值和期望值相等，执行更新value为50元。  
   此时小李给小明汇了50元，线程`3` expect=50元, update=100元，执行成功。  
   此时线程`1`恢复运行，取值value和expect相等，执行更新value为50元。。。。GG 小明莫名丢了50块钱。

   解决方案：加版本号，更新成功将值的版本号加1，

   更新前校验value==expectValue且version==exptctValue才执行更新操作。

3. 线程本地存储（Thread Local Storage）

4. 乐观读（StamptedLock.tryOptimisticRead()）在真正执行写操作前 validate(stamp)，不一致则升级为悲观读锁。

5. 原子类

6. Disruptor 有界内存队列

   - 内存分配更加合理（RingBuffer），数组元素在初始化时一次性全部创建 提升缓存命中率。
   - 对象循环利用，避免频繁GC。
   - 避免伪共享，提升缓存利用率。
   - 采用CAS方式设置入队索引。避免频繁加锁、解锁的性能消耗。
   - 支持批量消费，消费者可以无锁方式消费多个消息。

# 多线程

## 核心线程数如何设置

- CPU密集型的计算（很少进行IO请求）：corePoolSize = CPU核数。（+1，为了防止某个线程因内存失效或其他原因产生阻塞后还能有线程直接上来 保证CPU的利用率）
- IO密集型计算（系统IO请求比较频繁）：corePoolSize = CPU核数 * [ 1 + (I/O 耗时 / CPU耗时) ] 才能使CPU的利用率达到理论上的100%。如果耗时无法确定，可以先将corePoolSize设置为 2 * CPU核数 + 1。然后根据压测结果，灵活调整，已确定最佳线程数。

## 如何多线程步调一致

join

[多线程同步](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/threadpool/a0_多线程同步.java)：**CountDownLatch**、**CyclicBarrier**

## 并发工具类

简单的并行任务：[FutureTaskDemo](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/threadpool/b1_FutureTaskDemo.java)

任务之间有聚合关系：[CompletableFutureDemo](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/threadpool/b2_CompletableFutureDemo.java)

管理批量并行任务：[CompletionServiceDemo](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/threadpool/b3_CompletionServiceDemo.java)

**ForkJoinPool** 单机版的MapReduce [FockJoinDemo](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/threadpool/c1_FockJoinDemo.java)、[WordCountDemo](https://github.com/nibnait/algorithms/blob/master/src/main/java/jdk/concurrent/demo/threadpool/c2_FockJoinDemo2.java)