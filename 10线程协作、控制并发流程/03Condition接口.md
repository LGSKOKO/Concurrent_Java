# 03Condition接口

## 目录

- ### Condition接口介绍

- ### Condition接口的注意点

- ### 代码演示

------


## Condition接口介绍

- **Condition接口又称条件对象。**
- 当线程1需要等待某个条件的时候，它就去执行了condition.await()方法，一旦执行了await()方法，线程就会进入等待状态。
- 然后通常会有另外一个线程，假设是线程2，去执行对应的条件，直到这个条件达到的时候，线程2就会去执行condition.signal()方法，这时候JVM就会从被阻塞的线程里找，找到那些等待该condition的线程，然后线程1就会收到可执行信号的时候，它的线程状态就会变成Runnable可执行状态。
- **signalAll()和signal()区别**
- **signalAll()会唤起所有的等待线程。**
  - **signal()是公平的，会唤起等待时间最长的线程。**

## **Condition接口的注意点**

- **实际上，如果说Lock用来替代Synchronized，那么Condition就是用来替代对应的Object.wait/notify的，所以在用法和性质上面，几乎一样。**

- **await方法会自动释放持有的Lock锁，和Object.wait()方法一样会自动释放monitor（锁），不需要自己手动释放锁。**

- **调用await()的时候，必须持有锁，否则会抛出异常，和Object.wait()一样。**

## **代码演示**

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/20 10:31
   * @description：condition的基本使用
   */
  public class ConditionDemo1 {
      private ReentrantLock lock = new ReentrantLock();
      private Condition condition = lock.newCondition();
  
      void method1() throws InterruptedException {
          lock.lock();
          try {
              System.out.println("条件不满足，开始await");
              condition.await();
              System.out.println("条件满足了,开始执行后续任务");
          } finally {
              lock.unlock();
          }
      }
  
      void method2() {
          lock.lock();
          try {
              System.out.println("准备工作完成，开始唤醒");
              condition.signal();
          } finally {
              lock.unlock();
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
          ConditionDemo1 conditionDemo1 = new ConditionDemo1();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Thread.sleep(1000);
                      conditionDemo1.method2();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }).start();
  
          conditionDemo1.method1();
  
      }
  }
  
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/20 10:49
   * @description：实现生产者消费者
   */
  public class ConditionDemo2 {
      private int queueSize = 10;
      private PriorityQueue<Integer> queue = new PriorityQueue<>(queueSize);
  
      private ReentrantLock lock = new ReentrantLock();
      private Condition notFull = lock.newCondition();
      private Condition notEmpty = lock.newCondition();
  
      public static void main(String[] args) {
          ConditionDemo2 conditionDemo2 = new ConditionDemo2();
          Consumer consumer = conditionDemo2.new Consumer(conditionDemo2.queue);
          Produce produce = conditionDemo2.new Produce(conditionDemo2.queue);
  
          consumer.start();
          produce.start();
      }
  
      class Consumer extends Thread {
          private PriorityQueue queue;
  
          public Consumer(PriorityQueue queue) {
              this.queue = queue;
          }
  
          @Override
          public void run() {
              consumer();
          }
  
          private void consumer() {
              while (true) {
                  lock.lock();
                  try {
                      while (queue.size() == 0) {
                          System.out.println("队列空，等待新物品");
                          notEmpty.await();
                      }
                      queue.poll();
                      notFull.signalAll();
                      System.out.println("从队列里取走了一个物品，还剩" +
                              queue.size() + "个物品");
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
  
                      lock.unlock();
                  }
              }
          }
      }
  
      class Produce extends Thread {
          private PriorityQueue queue;
  
          public Produce(PriorityQueue queue) {
              this.queue = queue;
          }
  
          @Override
          public void run() {
              produce();
          }
  
          private void produce() {
              while (true) {
                  lock.lock();
                  try {
                      while (queue.size() == queueSize) {
                          System.out.println("队列满，等待物品被消费");
                          notFull.await();
                          System.out.println("开始生产新物品");
                      }
                      queue.add(1);
                      notEmpty.signalAll();
                      System.out.println("给队列里添加了一个物品，还剩" +
                              queue.size() + "个物品");
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      lock.unlock();
                  }
              }
  
          }
      }
      
  }
  ```

  

  

  