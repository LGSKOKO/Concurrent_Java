# 01原子类介绍

## 目录

- ### 什么是原子类，有什么作用？

- ### 6类原子类纵览

- ### Atomic*基本类型原子类，以AtomicInteger为例

- ### Atomic*Array数组类型原子类

- ### Atomic*Reference引用类型原子类

- ### **Atomic*FieldUpdater 升级类型原子类**

- ### Accumulator累加器

------

## 什么是原子类，有什么作用？

- **什么是原子类？**
  - **原子类中的操作是不可分割的，具有原子性。**
  - **在java.util.concurrent.atomic下。**
- **原子类有什么作用？**
  - **原子类的作用和锁类似，是为了保证并发情况下线程安全。**
  - **不过原子类相比于锁，有一定优势：**
  - **粒度更细：原子变量可以把竞争范围缩小到变量级别，这是我们可以获得的最细颗粒的情况了，通常锁的粒度都要大于原子变量的粒度。**
  - **效率更高：通常，使用原子类的效率会比使用锁的效率更高，除了高度竞争的情况。**

------

## 6类原子类纵览

| Atomic* 基本类型原子类                 |           AtomicInteger、AtomicLong、AtomicBoolean           |
| -------------------------------------- | :----------------------------------------------------------: |
| **Atomic*Array 数组类型原子类**        | **AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray** |
| **Atomic*Reference 引用类型原子类**    | **AtomicReference、AtomicStampedReference、AtomicMarkableReference** |
| **Atomic*FieldUpdater 升级类型原子类** | **AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater** |
| **Addater 累加器**                     |                  **LongAdder、DoubleAdder**                  |
| **Accumulator 累加器**                 |            **LongAccumulator、DoubleAccumulator**            |

------

## Atomic*基本类型原子类，以AtomicInteger为例

- **public final int get() //获取当前的值**

  **public fianl int getAndSet(int newValue) //获取当前的值，并设置新的值**

  **public final int getAndIncrement() //获取当前的值，并自增**

  **public final int getAndDecrement() //获取当前的值，并自减**

  **public final int getAndAdd(int delta) //获取当前的值，并加上预期值**

  **boolean compareAdnSet(int expect，int update) //如果当前的数 值等于预期值，则以原子方式将该值设置为输入值(update)**

-  [基本类型原子类代码](https://github.com/LGSKOKO/Concurrent_Java/blob/master/08atomic和CAS/02Atomic基本类型原子类代码.md) 

------

## Atomic*Array数组类型原子类

-  [数组类型原子类代码](https://github.com/LGSKOKO/Concurrent_Java/blob/master/08atomic和CAS/03AtomicArray数组类型原子类代码.md) 

## Atomic*Reference引用类型原子类

- **Atomic*Reference:AtomicReference类的作用 和AtomicInteger并没有本质区别，**

  **AtomicInteger可以让一个整数保证原子性，而AtomicReference可以让一个对象保证原子性；当然，AtomicReference的功能明显比AtomicInteger强，因为一个对象里可以包含很多属性。用法和AtomicInteger类似。**

-  [引用类型原子类代码](https://github.com/LGSKOKO/Concurrent_Java/blob/master/08atomic和CAS/04AtomicReference引用类型原子类代码.md) 

## **Atomic*FieldUpdater 升级类型原子类**

- **把普通变量升级为原子类：用AtomicIntegerFieldUpdater升级原有变量，对普通变量进行升级**。

- **使用场景：偶尔需要一个原子get-set操作。**

- **注意点：因为其背后 运用的原理是反射，所以要注意可见范围； 其不支持static的变量。**

-  [升级类型原子类代码](https://github.com/LGSKOKO/Concurrent_Java/blob/master/08atomic和CAS/05AtomicFieldUpdater升级类型原子类代码.md) 

## Adder累加器

- **是Java 8引入的，相对是比较新的一个类，高并发下LongAdder比AtomicLong效率高，不过本质是空间换时间。**

- **竞争激烈的时候，LongAdder把不同线程对应到不同的Cell上进行修改，降低了冲突的概率，是分段锁的理念，提高了并发性,ConcurrentHashMap也是多段锁的理念。**

-  [Adder累加器代码](https://github.com/LGSKOKO/Concurrent_Java/blob/master/08atomic和CAS/06Adder累加器代码.md) 

### **Adder累加器  总结：**

  -  **由于竞争很激烈，AtomicLong的实现原理是，每加一次 都要flush和refresh即进行同步，所以在高并发的时候冲突比较多，导致很耗费资源。**
  -   **AtomiLong在多线程的情况下，每次都需要同步，而LongAdder，每个线程都有自己的一个计数器，仅用来在自己线程内计数，这样一来就不会和其他线程的计数器干扰，多个线程都可以同步运行，最后调用sum方法时进行同步就可以了即将每个线程计算的结果进行同步加即可。**
  - **这就是LongAdder吞吐量比AtomicLong大的原因，本质是空间换时间。**
  -  **在低争用下，AtomicLong和LongAdder这两个具有相似的特征。但是在竞争激烈的情况下，LongAdder的吞吐量高得多，但需要更多的空间。**
  - **LongAdder适合的场景时统计和技术的场景，而且LongAdder基本只提供了add方法，而AtomicLong还具有coompareAndSet方法 即CAS**。

------

## Accumulator累加器

- **Accumulator和Adder非常相似，Accumulator就是一个更通用版本的Adder。**

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/12 14:12
   * @description：演示LongAccumulator
   *  适用场景：需要大量计算，并且可以并行计算的场景
   *  计算的顺序不能成为瓶颈
   */
  public class LongAccumulatorDemo {
      public static void main(String[] args) {
          //下面的lambda表达式可以根据场景 进行修改 如(x,y)->x+y y就是上一次x*y的值
          LongAccumulator accumulator =
                  new LongAccumulator((x,y)->x*y,1);
          ExecutorService executorService = Executors.newFixedThreadPool(8);
          IntStream.range(1,10).forEach(i->executorService.submit(
                  ()->accumulator.accumulate(i) ) );
          executorService.shutdown();
          while(!executorService.isTerminated()){}
          System.out.println(accumulator.get());//这是相当于LongAdder的sum方法 唯一同步
      }
  }
  ```

