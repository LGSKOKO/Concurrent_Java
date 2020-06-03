# 01wait、notify、norifyAll作用和用法

## **目录**

- **wait直到以下4种情况之一发生时，才会被唤醒**
- **notify唤醒**
- **代码展示**
- **wait/notify/notifyAll的特点、性质**

------

**wait作用**： **当线程执行wait()方法时候，会释放当前的锁，然后让出CPU，进入等待状态。** 

## wait直到以下4种情况之一发生时，才会被唤醒

- **另一个线程调用这个对象的notify()方法且刚好被唤醒的是本线程；**
- **另一个线程调用这个对象的notifyAll()方法；**
- **过了wait(long timeout)规定的超时时间，如果传入0就是永久等待；**
- **线程自身调用了interrupt()**

------

 **当 notify/notifyAll() 被执行时候，才会唤醒一个或多个正处于等待状态的线程，然后继续往下执行，直到执行完synchronized 代码块的代码或是中途遇到wait() ，再次释放锁。** 

## **notify唤醒**

- **notify方法只应该被拥有该对象的monitor的线程调用**
- **一旦线程被唤醒，线程便会从对象的“等待线程集合”中被移除，所以可以重新参与到线程调度当中**
- **要等刚才执行notify的线程退出被synchronized保护的代码并释放monitor**

------

## **代码展示**

- **普通方法**

```java
**
 * 描述：     展示wait和notify的基本用法 1. 研究代码执行顺序 2. 证明wait释放锁
 */
public class Wait {

    public static Object object = new Object();

    static class Thread1 extends Thread {

        @Override
        public void run() {
            synchronized (object) {
                System.out.println(Thread.currentThread().getName() + "开始执行了");
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程" + Thread.currentThread().getName() + "获取到了锁。");
            }
        }
    }

    static class Thread2 extends Thread {

        @Override
        public void run() {
            synchronized (object) {
                object.notify();
                System.out.println("线程" + Thread.currentThread().getName() + "调用了notify()");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread1 thread1 = new Thread1();
        Thread2 thread2 = new Thread2();
        thread1.start();
        Thread.sleep(400);
        thread2.start();
    }
}

```

- **notify和notifyAll展示**

```java
/**
 * 描述：     3个线程，线程1和线程2首先被阻塞，线程3唤醒它们。notify, notifyAll。 start先执行不代表线程先启动。
 */
public class WaitNotifyAll implements Runnable {

    private static final Object resourceA = new Object();//争夺的资源

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new WaitNotifyAll();
        Thread threadA = new Thread(r);
        Thread threadB = new Thread(r);
        Thread threadC = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resourceA) {
                    resourceA.notifyAll();//演示 notifyAll方法
//                    resourceA.notify(); //演示 notify方法
                    System.out.println("ThreadC notified.");
                }
            }
        });
        threadA.start();
        threadB.start();
       	Thread.sleep(400);
        threadC.start();
    }
    @Override
    public void run() {
        synchronized (resourceA) {
            System.out.println(Thread.currentThread().getName()+" got resourceA lock.");
            try {
                System.out.println(Thread.currentThread().getName()+" waits to start.");
                resourceA.wait();
                System.out.println(Thread.currentThread().getName()+"'s waiting to end.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```



- **只释放当前monitor展示**

```java
/**
 * 描述：     证明wait只释放当前的那把锁
 */
public class WaitNotifyReleaseOwnMonitor {

    private static volatile Object resourceA = new Object();
    private static volatile Object resourceB = new Object();

    public static void main(String[] args) {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resourceA) {
                    System.out.println("ThreadA got resourceA lock.");
                    synchronized (resourceB) {
                        System.out.println("ThreadA got resourceB lock.");
                        try {
                            System.out.println("ThreadA releases resourceA lock.");
                            resourceA.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }//end synchronized-B
                }//end synchronized-A
            }//end method run
        });

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (resourceA) {
                    System.out.println("ThreadB got resourceA lock.");
                    System.out.println("ThreadB tries to resourceB lock.");
                    synchronized (resourceB) {
                        System.out.println("ThreadB got resourceB lock.");
                    }//end synchronized-B
                }//end synchronized-A
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

------

## **wait/notify/notifyAll的特点、性质**

- **用必须先拥有monitor**   **(synchronized)**
- **只能唤醒其中一个,并且所唤醒的线程不受我们控制，即我们无法指定唤醒特定的线程(notify)**
- **属于Object类**
- **类似功能的有Condition，建议对比学习Condition**
- **同时持有多个锁的情况**   参照“**只释放当前monitor展示**”的Demo