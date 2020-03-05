# 05AtomicFieldUpdater升级类型原子类代码

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/2/12 11:14
 * @description： 演示AtomicIntegerFieldUpdater的用法
 */
public class AtomicIntegerFieldUpdaterDemo implements Runnable {
    static Candidate tom;
    static Candidate peter;
	//下面存入的是 clazz对象和对应的Field，注意和反射进行比照学习
    public static AtomicIntegerFieldUpdater<Candidate> scoreUpdater
            = AtomicIntegerFieldUpdater.newUpdater(Candidate.class,"score");

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            peter.score++;
            scoreUpdater.getAndIncrement(tom); //这里要传入的是对象实例，注意和反射进行比照学习
        }
    }

    static class Candidate{
        volatile int score;
    }

    public static void main(String[] args) throws InterruptedException {
        peter = new Candidate();
        tom = new Candidate();
        AtomicIntegerFieldUpdaterDemo r = new AtomicIntegerFieldUpdaterDemo();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("普通变量:"+peter.score);
        System.out.println("升级后的变量："+tom.score);
    }
}
```

