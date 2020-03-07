# 02ConcurrentHashMap

## 目录

- ### **Map关系图**

- ### **为什么HashMap是不安全的**

- ### **HashMap关于并发的特点**

- ### ConcurrentHashMap

## **Map关系图**

![Map关系图](https://raw.github.com/LGSKOKO/Concurrent_Java/master/09并发容器/img/Map关系图.jpg)

## **为什么HashMap是不安全的**

- **同时put碰撞导致数据丢失。**
  - 多个线程同时put并且key的hash值一样，所以会导致一个线程的数据会被覆盖。
- **同时put扩容导致数据丢失  **
  - 多个线程同时发现需要扩容，同时进行扩容，所以会导致最后只有一个线程扩容的数组被保存下来。
- **扩容时 可能会造成循环引用，所以会死循环造成的CPU100%**

------

## HashMap关于并发的特点

- **非线程安全的**
- **迭代时不允许修改**
- **只读的并发时安全的**
- **如果一定要把HashMap用在并发环境中，用Collections.synchronizedMap(new HashMap() )**

------

## ConcurrentHashMap

### 1.7中ConcurrentHashMap的实现和分析

- Java7中的最外层是多个segement，每个segment的底层数据结构与HashMap类似，仍然是数组和链表组成的拉链法。
- 每个segment独立上ReentrantLock锁，每个segment之间互不影响，提高了并发效率。
- 默认有16个Segments，所以最多可以同时支持16个线程并发写（操作分别分布在不同的Segment上)。这个默认值可以在初始化的时候设置为其他值，但是一旦初始化以后，是不可扩容的。
- ![Map关系图](https://raw.github.com/LGSKOKO/Concurrent_Java/master/09并发容器/img/Java7CHM.jpg)

### 1.8中ConcurrentHashMap的实现和分析

- **putVal流程**
- 判断 key value不为空 -> 计算hash值 -> 根据对应位置结点的类型 来赋值，或者helpTransfer，或者增长链表，或者给红黑树增加结点 -> 检查满足阈值就 ”红黑树化“ ，链表变红黑树的阈值是8-> 返回oldVal
  
- **get流程**

  - 计算hash值 -> 直接找到对应的位置，根据情况进行：直接取值 | 红黑树里取值 |遍历链表取值 ->返回找到的结果
  - ![Map关系图](https://raw.github.com/LGSKOKO/Concurrent_Java/master/09并发容器/img/Java8CHM.jpg)

### 对比1.7和1.8，为什么要把1.7的结构改成1.8的结构

- **数据结构的区别：**这样做提高了并发性，java7采用的是segment结构，其中锁的个数是并发度，java8中每个节点都能并发；
-   **hash碰撞：** java7中碰撞是拉链法，java8中开始是用拉链法，当达到阈值8时采用红黑树。
-   **保证并发安全：**1.7中才用的是分段锁，用的是segment而segment继承自ReentrantLock，java8中使用的是cas加上synchronized。
- **查询复杂度：** java7链表查询是O(n) java8中如果是红黑树查询复杂度是O(log2n)

  





  



































