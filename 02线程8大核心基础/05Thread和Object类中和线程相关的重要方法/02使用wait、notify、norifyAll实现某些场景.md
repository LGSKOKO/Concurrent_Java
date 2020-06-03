# 02使用wait、notify、norifyAll实现某些场景

## 目录

- **手写生产者消费者设计模式**
- **用程序实现两个线程交替打印 0~100 的奇偶数**
- **常见面试问题**

------

**手写生产者消费者设计模式**

```java

/**
 * 描述：     用wait/notify来实现生产者消费者模式
 */
public class ProducerConsumerModel {
    public static void main(String[] args) {
        EventStorage eventStorage = new EventStorage();
        Producer producer = new Producer(eventStorage);
        Consumer consumer = new Consumer(eventStorage);
        new Thread(producer).start();
        new Thread(consumer).start();
    }
}

class Producer implements Runnable {

    private EventStorage storage;

    public Producer(
            EventStorage storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            storage.put();
        }
    }
}

class Consumer implements Runnable {

    private EventStorage storage;

    public Consumer(
            EventStorage storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            storage.take();
        }
    }
}

class EventStorage {

    private int maxSize;
    private LinkedList<Date> storage;

    public EventStorage() {
        maxSize = 10;
        storage = new LinkedList<>();
    }

    public synchronized void put() {
        while (storage.size() == maxSize) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        storage.add(new Date());
        System.out.println("仓库里有了" + storage.size() + "个产品。");
        notify();
    }

    public synchronized void take() {
        while (storage.size() == 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("拿到了" + storage.poll() + "，现在仓库还剩下" + storage.size());
        notify();
    }
}
```

------

## 用程序实现两个线程交替打印 0~100 的奇偶数**

​	**偶线程：0 奇线程：1 偶线程:2 ......**

- **基本思路：synchronized**

  ```java
  /**
   * 描述：     两个线程交替打印0~100的奇偶数，用synchronized关键字实现  会有无效的操作
   */
  public class WaitNotifyPrintOddEvenSyn {
  
      private static int count;
  
      private static final Object lock = new Object();
  
      //新建2个线程
      //1个只处理偶数，第二个只处理奇数（用位运算）
      //用synchronized来通信
      public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  while (count < 100) {
                      synchronized (lock) {
                          if ((count & 1) == 0)  //位运算 
                              System.out.println(Thread.currentThread().getName() + ":" + count++); 
                      }
                  }
              }
          }, "偶数").start();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  while (count < 100) {
                      synchronized (lock) {
                          if ((count & 1) == 1) //位运算
                              System.out.println(Thread.currentThread().getName() + ":" + count++); 
                      }
                  }
              }
          }, "奇数").start();
      }
  }
  ```

  

- **更好的方法：wait/notify**

  ```java
  /**
   * 描述：     两个线程交替打印0~100的奇偶数，用synchronized关键字实现
   */
  public class WaitNotifyPrintOddEvenSyn {
  
      private static int count;
  
      private static final Object lock = new Object();
  
      //新建2个线程
      //1个只处理偶数，第二个只处理奇数（用位运算）
      //用synchronized来通信
      public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  while (count < 100) {
                      synchronized (lock) {
                          if ((count & 1) == 0) {
                              System.out.println(Thread.currentThread().getName() + ":" + count++);
                          }
                      }
                  }
              }
          }, "偶数").start();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  while (count < 100) {
                      synchronized (lock) {
                          if ((count & 1) == 1) {
                              System.out.println(Thread.currentThread().getName() + ":" + count++);
                          }
                      }
                  }
              }
          }, "奇数").start();
      }
  }
  ```

  ------

  ## **常见面试问题**

  - **手写生产者消费者设计模式 （重中之重)**
  - **为什么wait() 需要在同步代码块内使用，而 sleep() 不需要**
  - **为什么线程通信的方法wait(), notify()和notifyAll()被定义在Object类里？而sleep定义在Thread类里？**
  - **wait方法是属于Object对象的，那调用Thread.wait会怎么样？**
  - **如何选择用notify还是nofityAll？**
  - **notifyAll之后所有的线程都会再次抢夺锁，如果某线程抢夺失败怎么办？**
  - **用suspend()和resume()来阻塞线程可以吗？为什么？**