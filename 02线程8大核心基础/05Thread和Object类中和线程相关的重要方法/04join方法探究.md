# 04join方法探究

## 目录

- **作用、用法**
- **注意的点**
- **常见面试问题**

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

- **join方法的源码也是使用了wait**

  

------

## **常见面试问题**

**在join期间，线程处于哪种线程状态？**

**答：WAITTING**