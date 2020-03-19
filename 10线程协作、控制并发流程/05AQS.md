# 05AQS（进阶必备，并发灵魂）

## 目录

- ### **学习AQS的思路**

- ### 为什么需要学习AQS

- ### AQS内部原理解析

- ### **AQS的应用实例**

## **学习AQS的思路**

1. **学习AQS的主要目的是理解原理、提高技术、以及应对面试。**
2. **先从应用层面理解为什么需要它 如何使用它，然后在看一看Java代码的设计者是如何使用它的 了解它的应用场景。**
3. **这样之后我们再去分析它的结构，我们就学习得更加轻松了。**

## 为什么需要学习AQS

- **我们以及学习过了ReentrantLock和Semaphore，可以发现它们有很多相似点。**
- **事实上，不仅是ReenrantLock和Semaphore，包括CountDownLatch、ReentrantReadWriteLock都有这样类似的“协作”（或者叫“同步”）功能，其实它们底层都共同使用了一个共同的基类 就是AQS。**
- **例如：Semaphore内部有一个Sync类，Sync类继承了AQS；上面的那些类的内部也都有一个Sync类**
- **AQS全称是：AbstractQueuedSynchronizer**
- **AQS：因为上面的协作类，它们有很多工作都是类似的，所以如果能提取出一个工具类 可以直接用，对于ReentrantLock和Semaphore而言就可以屏蔽很多细节，只要关注它们自己的“业务逻辑”就可以了。**

------

## **AQS内部原理解析**

- ### **state**

  - **这里的state的具体含义，会根据具体实现类的不同而不同，比如在Semaphore里，它表示“剩余的许可证的数量”，而在CountDownLatch里，它表示“还需要倒数的数量”。**
  - **state是volatile修饰的，会被并发地修改，所以所有修改state的方法都需要保证线程安全，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于juc.atomic包的支持。**
  - **在ReentrantLock中，state用来表示“锁”的占有情况，包括可重入计数，当state的值为0的时候，标识改Lock不被任何线程占有。**

- ### **控制线程抢锁和配合的FIFO队列**

  - **这个队列用来存放 “等待的线程”，AQS就是“排队管理器”，当多个线程争用一把锁时，必须有排队机制将那些没能拿到锁的线程串在一起。当锁释放的时候，锁管理器就会挑一个合适的线程来占有这个刚刚释放的锁。**
  - **AQS会维护一个等待的线程队列，把线程队列放到这个队列里。**
  - **这是一个双向形式的队列，基于双向链表实现。**

- ### **期望协作工具类去实现的获取/释放等重要方法。**

  - **这里的获取和释放方法，是利用AQS的协作类里最重要的方法，是由协作类自己去实现的，并且含义各不相同。**

-------

## **AQS的应用实例**

- ### **AQS用法**

  - **第一步：写一个类，想好协作的逻辑，实现获取/释放方法。**
  - **第二步：内部写一个Sync类继承 AbstractQueuedSynchronizer。**
  - **第三步：根据是否独占来 重写tryAcquire/tryRelease（独占）或tryAcquireShared(int acquires)和tryReleaseShared(int releases)等方法（共享），在之前写的获取和释放方法中调用 AQS的acquire/release或Shared方法。**

  

  

  

  
