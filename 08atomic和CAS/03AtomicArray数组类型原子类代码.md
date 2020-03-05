# 03AtomicArray数组类型原子类代码

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/2/12 6:40
 * @description：演示原子数组的使用方法
 * @modified By：
 * @version: $
 */
public class AtomicArrayDemo {

    public static void main(String[] args) {
        AtomicIntegerArray arrays = new AtomicIntegerArray(1000);
        Thread[] inThreads = new Thread[1000];
        Thread[] deThreads = new Thread[1000];

        Incrementer incrementer = new Incrementer(arrays);
        Decrementer decrementer = new Decrementer(arrays);

        for (int i = 0; i < inThreads.length; i++) {
            inThreads[i] = new Thread(incrementer);
            deThreads[i] = new Thread(decrementer);
        }
        for (int i = 0; i < inThreads.length; i++) {
            inThreads[i].start();
            deThreads[i].start();
        }
        for (int i = 0; i < arrays.length(); i++) {
            try {
                inThreads[i].join();
                deThreads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        for (int i = 0; i < arrays.length(); i++) {
            if (arrays.get(i) != 0)
                System.out.println("下标"+i + "的值不为0");
        }
        System.out.println("运行结束");
    }
}

class Incrementer implements Runnable {
    private AtomicIntegerArray arrays;

    public Incrementer(AtomicIntegerArray arrays) {
        this.arrays = arrays;
    }

    @Override
    public void run() {
        for (int i = 0; i < arrays.length(); i++) {
            arrays.getAndIncrement(i);
        }
    }
}

class Decrementer implements Runnable {
    private AtomicIntegerArray arrays;

    public Decrementer(AtomicIntegerArray arrays) {
        this.arrays = arrays;
    }

    @Override
    public void run() {
        for (int i = 0; i < arrays.length(); i++) {
            arrays.getAndDecrement(i);
        }
    }
}

```

