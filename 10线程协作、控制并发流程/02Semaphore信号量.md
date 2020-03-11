# 02Semaphore信号量

## 目录

- ### Semaphore信号量介绍

- ### Semaphore信号量的使用流程

- ### Semaphore信号量主要方法介绍

- ### Semaphore信号量的注意点

- ### 代码演示

------

## Semaphore信号量介绍

- **Semaphore可以用来限制或管理数量有限的资源的使用情况。**
- 信号量的作用是维护一个“许可证”的计数，线程可以”获取“许可证，线程“获取”许可证后，那么信号量剩余的许可证就减一；线程也可以“释放”一个许可证，那么信号量剩余的许可证就加一，当信号量所拥有的的许可证数量为0，那么下一个还想要获取许可证的线程，就需要等待，直到有另外的线程释放了许可证。

## Semaphore信号量的使用流程

1. **初始化Semaphore并指定许可证的数量。**
2. **在需要许可证的代码前加acquire()或者acquireUninterruptibly()方法。**
3. **在任务结束后，调用release()来释放许可证。**

## Semaphore信号量主要方法介绍

- **new Semaphore(int permits,boolean fair) 这里可以设置是否需要使用 公平策略，如果传入true，那么是公平的。如果是公平策略，Semaphore会把之前等待的线程放到FIFO队列里，以便于 当有了新的许可证，可以分发给之前等了最长时间的线程。 **
- **acquire()，获取许可证，获取不到 会陷入阻塞。**
- **tryAcquire():尝试获取许可证，会立即返回获取结果，不会陷入阻塞。**
- **tryAcquire(timeout) :可以设置超时时间，这个时间内尝试获取许可证，获取不到继续执行别的事情。**
- **release()：释放许可证。**

## Semaphore信号量的注意点

1. **获取和释放的许可证数量必须一致，否则随着时间的推移，最后许可证数量不够用，会导致线程卡死。**
2. **注意在初始化Semaphore的时候设置公平性，一般设置为true比较合理。**
3. **并不是必须由获取许可证的线程 来释放许可证，事实上，获取和释放许可证对线程并无要求，也可以是A线程获取，B线程释放，只要逻辑合理即可。**
4. **信号量的作用，除了控制临界区最多同时有N个线程访问外，另一个作用是可以实现“条件等待”，例如线程1需要在线程2完成准备工作后才能开始工作，那么就线程1 先acquire，然后线程2 完成任务后 release()，这样的话，相当于是轻量级的CountDownLatch()。**

------

## 代码演示

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/20 9:38
   * @description：
   */
  public class SemaphoreDemo {
  
      private static Semaphore semaphore = new Semaphore(3);//这里没有传入true，默认是非公平的
      public static void main(String[] args) {
          ExecutorService service = Executors.newFixedThreadPool(50);
          for (int i = 0; i < 100; i++) {
              service.submit(new Task());
          }
          service.shutdown();
  
      }
  
      static class Task implements Runnable{
          @Override
          public void run() {
              try {
                  semaphore.acquire();
                  //下面的语句是 此线程得同时获取到两个许可证才能运行
  //                semaphore.acquire(2);
                  System.out.println(Thread.currentThread().getName()+
                          "拿到了许可证");
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              try {
                  Thread.sleep(500);
              }catch (Exception e) {
                  e.printStackTrace();
              } finally{
                  System.out.println(Thread.currentThread().getName()+
                          "释放了许可证");
                  semaphore.release();
                  //如果前面使用了semaphore.acquire(2)
                  //这里就要使用 semaphore.release(2); 获取和释放量要一致
              }
          }
      }
  
  }
  
  ```

  
