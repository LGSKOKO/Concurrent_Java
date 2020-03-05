# 06Adder累加器代码

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/2/12 11:42
 * @description：演示LongAdder性能比AtomicLong高
 */
public class AtomicLongDemo {

    static class Task implements Runnable {
        private AtomicLong count;

        public Task(AtomicLong count) {
            this.count = count;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                count.getAndIncrement();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(20);
        AtomicLong count = new AtomicLong(0);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            executorService.submit(new Task(count));
        }
        //不能再提交任务 并等待所有任务执行完成
        executorService.shutdown();
        //循环等待所有任务执行完成
        while (!executorService.isTerminated()) {
        }
        long end = System.currentTimeMillis();
        System.out.println("count的值为" + count); 
        long time = end - start;
        System.out.println("AtomicLong所需时间为" + time);
    }
}
```

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/2/12 11:42
 * @description：演示LongAdder性能比AtomicLong高
 */
public class LongAdderDemo {

    static class Task implements Runnable {
        private LongAdder count;
        public Task(LongAdder count) {
            this.count = count;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                count.increment();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(20);
        LongAdder count = new LongAdder();
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            executorService.submit(new Task(count));
        }
        //不能再提交任务 并等待所有任务执行完成
        executorService.shutdown();
        //循环等待所有任务执行完成
        while (!executorService.isTerminated()) {
        }
        long end = System.currentTimeMillis();
        System.out.println("count的值为" + count.sum());//进行求和 这是唯一需要同步的地方
        long time = end - start;
        System.out.println("LongAdder所需时间为" + time);
    }
}

```

