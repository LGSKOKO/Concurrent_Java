# 07CAS

## 目录

- ### 什么是CAS?

- ### 案例演示

- ### 应用场景

- ### 以AtomicInterger为例，分析Java中如何利用CAS实现原子操作

- ### 缺点

------

## 什么是CAS?

-  **CAS是compare and swap的缩写，即我们所说的比较交换。CAS是一种基于锁的操作，而且是乐观锁 。**

- **CAS有三个操作数，内存值V、预期值A、要修改的值B；**

  **并且只有当预期值A和内存值V相同时，才能将内存值修改为B；**

  **否则什么都不做，最后返回现在的V值。**

------

## 案例演示

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/2/13 11:57
 * @description：
 */
public class TwoThreadsCompetition implements Runnable {
    private volatile int value = 0;

    public synchronized int compareAndSet(int expect,int newValue)
    {
        int oldValue = value;
        if(oldValue == expect)
        {
            value = newValue;
        }
        return oldValue;
    }
    @Override
    public void run() {
        compareAndSet(0,1);
    }

    public static void main(String[] args) throws InterruptedException {
        TwoThreadsCompetition r = new TwoThreadsCompetition();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);

        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(r.value);
    }
}
```

------

## 应用场景

- **乐观锁**
- **并发容器类**
- **原子类**

## 以AtomicInterger为例，分析Java中如何利用CAS实现原子操作

- **首先，AtomicInteger加载Unsafe工具，用来直接操作内存数据。**

- **然后用Unsafe来实现底层操作，主要是Unsafe类中的compareAndSwapInt方法，方法中先拿到变量value在内存中的地址。**

- **重点是，用volatile修饰value字段，保证可见性。**

- **补充，Unsafe是CAS的核心类，Java无法直接访问底层操作系统，而是通过本地（native）方法进行访问。不过尽管如此，JVM还是提供了一个后门，JDK中有一个Unsafe类，它提供了硬件级别的原子操作。**

- ```java
   //下面是AtomicInteger类的代码
  // setup to use Unsafe.compareAndSwapInt for updates
      private static final Unsafe unsafe = Unsafe.getUnsafe();
      private static final long valueOffset;
  
      static {
          try {
              valueOffset = unsafe.objectFieldOffset	
                  (AtomicInteger.class.getDeclaredField("value"));
              //这里使用了unsafe工具 并通过反射获取到value字段
          } catch (Exception ex) { throw new Error(ex); }
      }
  
      private volatile int value;//用volatile修饰
  
  
   /**
       * Atomically adds the given value to the current value.
       *
       * @param delta the value to add
       * @return the previous value
       */
      public final int getAndAdd(int delta) {  //调用的是unsafe的方法。
          return unsafe.getAndAddInt(this, valueOffset, delta);
      }
  
  ```

- ```java
  //下面的是unsafe对应的方法 调用的是do while的循环
  public final int getAndAddInt(Object var1, long var2, int var4) {
          int var5;
          do {
              var5 = this.getIntVolatile(var1, var2);
          } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  
          return var5;
      }
  ```

------

## **缺点**

- **会有ABA问题；一个线程a将数值改成了b，接着又改成了a，此时CAS认为是没有变化，其实是已经变化过了，而这个问题的解决方案可以使用版本号标识，每操作一次version加1，数据库也是用视图解决此类问题的。在java5中，已经提供了AtomicStampedReference来解决问题。 **
- **自旋时间过长； 之前说过了CAS里面是一个循环判断的过程，如果线程一直没有获取到状态，cpu资源会一直被占用。 **