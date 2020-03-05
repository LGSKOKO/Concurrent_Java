# 01Lock接口

## 目录

- ### **简介、地位、作用**

- ### **为什么synchronized不够用，为什么需要Lock？**

- ### **方法介绍**

- ### **代码演示**

- ### 可见性保证

------

## Lock简介、地位、作用

- **锁是一种工具，用于控制对共享资源的访问。**
- **Lock和synchronized，这两个是最常见的锁，他们都可以达到线程安全的目的，但是使用上和功能上又有较大的不同。**
- **Lock并不是 用来替代synchronized的，而是当使用synchronized不适合或不满足要求的时候，来提供高级功能的。**
- **Lock接口最常见的实现类是ReentrantLock。**
- **通常情况下，Lock只允许一个线程来访问这个共享资源。不过有的时候，一些特殊的实现也可允许并发访问，比如ReadWriteLock里面的ReadLock。**

------

##   为什么synchronized不够用，为什么需要Lock？

- **为什么synchronized不够用?**
  - **1）效率低：所得所得释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程。**
  - **2）不够灵活(读写锁更灵活)：加锁和释放锁的时机单一，每个锁仅有单一的条件(某个对象),可能是不够的的。**
  - **3）无法知道是否成功获取到锁。**

------

##   **方法介绍**

- **在Lock中生命力四个方法来获取锁**
  - **lock()、tryLock()、tryLock(long time，TimeUnit unit)和 lockInterruptibly()。**
- **那么这四个方法有啥区别呢？**
  -  **lock()就是最普通的获取锁，如果锁已经被其他线程获取，则进行等待。**
  - **Lock不会像synchronized一样在异常时自动释放锁。即synchronized发生异常是会自动释放锁**
  - **因此最佳实践是，在finally中释放锁，以保证发生异常时锁一定被释放。**
  - **lock()方法不能陷入中断，这会带来很大的隐患：一旦陷入死锁,lock()就会陷入永久等待。**
- **tryLock()**
  - **tryLock()用来尝试获取锁，如果当前锁没有被其他线程占用，则获取成功，返回true，否则返回false，代表获取锁失败。**
  - **相比于lock()，这样的方法显然功能更强大了，我们可以根据是否能获取到锁来决定后续程序的行为。**
  - **该方法会立即返回，即拿不到锁也立刻返回 不会等待获取锁。**
- **tryLock(long time，TimeUnit unit)：超时就放弃；该方法也是可以中断的，返回值为boolean。**
- **lockInterruptibly()：相当于tryLock(long time，TimeUnit unit)把超时时间设置为无限。在等锁的过程中，线程可被中断。**
- **unlock()：解锁。**

------

##  **代码演示**

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/11 11:29
   * @description：用tryLock来避免死锁
   */
  public class TryLockDeadlock implements Runnable {
      
      int flag = 1;
      static Lock lock1 = new ReentrantLock();
      static Lock lock2 = new ReentrantLock();
  
      public static void main(String[] args) {
          TryLockDeadlock r1 = new TryLockDeadlock();
          TryLockDeadlock r2 = new TryLockDeadlock();
          r1.flag = 1;
          r1.flag = 0;
          new Thread(r1).start();
          new Thread(r2).start();
  
      }
  
      @Override
      public void run() {
          for (int i = 0; i < 100; i++) {
              if (flag == 1) {
                  try {
                      if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程1获取到了锁1");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock2.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程1获取到了锁2");
                                      System.out.println("线程1成功获取到了两把锁");
                                      break;
                                  } finally {
                                      lock2.unlock();
                                  }
                              } else {
                                  System.out.println("线程1获取锁2失败，已重试");
                              }
                          } finally {
                              lock1.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程1获取锁1失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
  
              if (flag == 0) {
                  try {
                      if (lock2.tryLock(3000, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程2获取到了锁2");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程2获取到了锁1");
                                      System.out.println("线程2成功获取到了两把锁");
                                      break;
                                  } finally {
                                      lock1.unlock();
                                  }
                              } else {
                                  System.out.println("线程2获取锁1失败，已重试");
                              }
                          } finally {
                              lock2.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程2获取锁2失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
  
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/11 12:02
   * @description：演示LockInterruptibly()
   */
  public class LockInterruptibly implements Runnable {
      Lock lock = new ReentrantLock();
  
      public static void main(String[] args) {
          LockInterruptibly lock = new LockInterruptibly();
          Thread thread0 = new Thread(lock);
          Thread thread1 = new Thread(lock);
          thread0.start();
          thread1.start();
          try {
              Thread.sleep(2000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          //可以尝试将下面两个语句分别注释，查看输出效果
  //        thread0.interrupt();
          thread1.interrupt();
      }
      @Override
      public void run() {
          try {
              lock.lockInterruptibly();
              try{
                  System.out.println(Thread.currentThread().getName()+"获取到了锁");
                  Thread.sleep(5000);
              }catch (Exception e){
                  System.out.println(Thread.currentThread().getName()+"休眠期间被打断了");
              } finally{
                  System.out.println(Thread.currentThread().getName()+"释放了锁");
                  lock.unlock();
              }
          }catch (Exception e){
              System.out.println(Thread.currentThread().getName()+"等锁期间被打断了");
          }
      }
  }
  
  ```

------

## 可见性保证

- **可见性**
- **happens-befor原则**
- **Lock的加解锁和synchronized具有同样内存语义，也就是说，下一个加锁前可以看到所有前一个解锁后发生的所有语句。**
-  **[内存可见性参考](https://github.com/LGSKOKO/Concurrent_Java/blob/master/03Java内存模型/03内存可见性/01内存可见性问题.md)** 