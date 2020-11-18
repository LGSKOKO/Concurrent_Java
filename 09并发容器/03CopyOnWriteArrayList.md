# 03CopyOnWriteArrayList

- ### **CopyOnWriteArrayList诞生记**

- ### **CopyOnWriteArrayList适合场景**

- ### **CopyOnWriteArrayList读写规则**

- ### CopyOnWriteArrayList缺点

------

## **CopyOnWriteArrayList诞生记**

- **替代Vector和SynchronizedList 就和ConcurrentHashMap替代SynchronizedMap的原因一样。**
- **Vector和SynchronizedList的锁的粒度太大，并发效率相比比较低，并且迭代时无法编辑。**
- **Copy-On-Write并发容器还包括CopyOnWriteArraySet，用来替代同步Set。**

##  **CopyOnWriteArrayList适合场景**

- **读操作尽可能地快，而写即使慢一些也没有太大关系。**
- **读多写少的场景。**

## CopyOnWriteArrayList读写规则

- **读写锁：读读共享、其他互斥（读写互斥、写读互斥、写写互斥）。**

- **读写锁规则的升级：读取完全不用加锁，并且更厉害的是写入也不会阻塞读取操作。只有写入和写入之间需要进行同步等待。**

- ```java
      /**
       * Appends the specified element to the end of this list.
       *
       * @param e element to be appended to this list
       * @return {@code true} (as specified by {@link Collection#add})
       */
      public boolean add(E e) {
          final ReentrantLock lock = this.lock;
          lock.lock();//这里用锁 进行限制多个线程同时写问题
          try {
              Object[] elements = getArray();
              int len = elements.length;
              Object[] newElements = Arrays.copyOf(elements, len + 1);//复制一个新的数组
              newElements[len] = e;//将元素添加到 新数组的最后一位
              setArray(newElements);
              return true;
          } finally {
              lock.unlock();
          }
      }
  ```

## CopyOnWriteArrayList缺点

- **数据不一致问题：CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入时的数据，马上能读到，请不要用CopyOnWrite容器。**

- **内存占用问题：因为CopyOnWrite是写时复制，所以在写操作的时候，内存会同时驻扎两个对象的内存。**

  
