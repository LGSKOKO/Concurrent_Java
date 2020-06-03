# 04join方法探究

## 目录

- **作用、用法**
- **注意的点**
- **常见面试问题**
- **相关博客**

------

## 作用、用法

```java
/**
 * 描述：     演示join，注意语句输出顺序，会变化。
 作用：因为新的线程加入了我们，所以我们要等他执行完再出发
 用法：main等待thread1执行完毕，注意谁等谁
 */
public class Join {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "执行完毕");
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "执行完毕");
            }
        });

        thread.start();
        thread2.start();
        System.out.println("开始等待子线程运行完毕");
        thread.join();
        thread2.join();
        System.out.println("所有子线程执行完毕");
    }
}

```

------

## **注意的点**

- **join与CountDownLatch或CyclicBarrier的区别**

- **join方法的源码也是使用了wait()**

- ```java
//join方法源码
  //方法join(long)的功能在内部是使用wait(long)来实现的，所以join(long)方法具有释放锁的特点。
  public final void join() throws InterruptedException {
          join(0);
      }
      public final synchronized void join(long millis)
      throws InterruptedException {
          long base = System.currentTimeMillis();
          long now = 0;
  
          if (millis < 0) {
              throw new IllegalArgumentException("timeout value is negative");
          }
  
          if (millis == 0) {
              while (isAlive()) {
                  wait(0);
              }
          } else {
              while (isAlive()) {
                  long delay = millis - now;
                  if (delay <= 0) {
                      break;
                  }
                  wait(delay);
                  now = System.currentTimeMillis() - base;
              }
          }
      }
  /*
  join方法中如果传入参数，则表示这样的意思：如果A线程中掉用B线程的join(10)，则表示A线程会等待B线程执行10毫秒，10毫秒过后，A、B线程并行执行。需要注意的是，jdk规定，join(0)的意思不是A线程等待B线程0秒，而是A线程等待B线程无限时间，直到B线程执行完毕，即join(0)等价于join()。(其实join()中调用的是join(0))
  */
  ```

------

## **常见面试问题**

**在join期间，线程处于哪种线程状态？**

**答：WAITTING**

------

## 相关博客

- https://www.iteye.com/blog/uule-1101994