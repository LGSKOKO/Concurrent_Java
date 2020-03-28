# 02volatile关键字

## 目录

- **volatile是什么？**
- **volatile的适用场景和不适用场景**
- **volatile的两点作用**
- **volatile和synchronized的关系？**
- **volatile小结**

------

## **volatile是什么？**

- **volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用volatile并不会发生上下文切换等开销很大的行为。**
- **如果一个变量别修饰成volatile，那么JVM就知道了这个变量可能会被并发修改。然后不会进行重排序等操作**
- **但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的原子保护，volatile仅在很有限的场景下才能发挥作用。**

------

## volatile的适用场景和不适用场景

- **不适用场景：** a++等场景，即不适用于组合操作
- **适用场景：**
  - **纯赋值操作** ，如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。
  - **作为刷新之前变量的触发器**，用了volatile int x后，可以保证读取x后，之前的所有变量都可见。

**不适用场景代码：**

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/1/21 15:07
 * @description：volatile不适用场景
 * @modified By：
 * @version: $
 */
public class NoVolatile implements Runnable {
     volatile int a = 0;
    AtomicInteger realA = new AtomicInteger();
    public static void main(String[] args) throws InterruptedException {
        NoVolatile v = new NoVolatile();
        Thread thread = new Thread(v);
           Thread thread2 = new Thread(v);
        thread.start();
        thread2.start();
        thread.join();
        thread2.join();
        System.out.println(v.a);
        System.out.println(v.realA);
    }
    @Override
    public void run() { 
        for (int i = 0; i < 20000; i++) {
            a++;
            realA.incrementAndGet();
        }
    }
}

```

------

## **volatile的两点作用**

- **第一层 可见性**，读一个 volatile 变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值，写一个 volatile 属性会立即刷入到主内存。
- **第二层 禁止指令重排序优化**，解决单例双重锁乱序问题。

## volatile和synchronized的关系？

**volatile在这方面可以看做是轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。**

------

## **volatile小结**

- volatile 修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag；或者作为触发器，实现轻量级同步。
- volatile 属性的读写操作都是无锁的，它不能替代 synchronized，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的。
- volatile 只能作用于属性，我们用 volatile 修饰属性，这样 compilers 就不会对这个属性做指令重排序。
- volatile 提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile 属性不会被线程缓存，始终从主存中读取。
- volatile 提供了 happens-before 保证，对 volatile 变量 v 的写入 happens-before 所有其他线程后续对 v 的读操作。
- volatile 可以使得 long 和 double 的赋值是原子的。  对于64位的long和double，如果没有被volatile修饰，那么对其操作可以不是原子的。在操作的时候，可以分成两步，每次对32位操作。  