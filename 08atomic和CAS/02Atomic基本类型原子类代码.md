# 02Atomic基本类型原子类代码

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/2/12 6:18
 * @description：  演示原子类的使用方法
 */
public class AtomicIntegerDemo1 implements Runnable {
    private static AtomicInteger atomicInteger = new AtomicInteger();
    private static volatile int  basicCount = 0;

    public void incrementAtomic(){
        atomicInteger.getAndIncrement();
        //atomicInteger.getAndAdd(-10);
    }
    public void incrementBasic(){
        basicCount++;
    }
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
              incrementAtomic();
              incrementBasic();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerDemo1 r = new AtomicIntegerDemo1();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("原子类的值是"+atomicInteger.get());
        System.out.println("普通变量的值是"+basicCount);
    }
}
```

