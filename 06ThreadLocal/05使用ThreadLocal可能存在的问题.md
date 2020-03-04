# 05使用ThreadLocal可能存在的问题

## 目录

- ### **内存泄露问题**

- ### **NPE问题**

- ### **补充**

-------

## **内存泄露问题**

- **什么是内存泄露：某个对象不再有用，但是占的内存却不能被收回。**

- ```java
  //ThreadLocal类的 内部类ThreadLocalMap中的 内部类Entry的源码
  //通过源码 我们可知Entry的key 是弱引用，value是强引用
  //Java中一共存在四种引用类型：强引用、软引用、弱引用、虚引用
  //弱引用的特点是，如果这个对象只被弱引用关联（没有任何强引用关联），那么这个对象就可以被回收
  //所以弱引用不会阻止GC，因此这个弱引用的机制不会引发内存泄露
  static class Entry extends WeakReference<ThreadLocal<?>> {
              /** The value associated with this ThreadLocal. */
              Object value;
  
              Entry(ThreadLocal<?> k, Object v) {
                  super(k);
                  value = v;
              }
          }
  ```

- **正常情况下，当线程终止，保存在ThreadLocal里的value会被垃圾回收，因为没有任何的强引用了。**

- **但是，如果下次爱你成不能终止（比如线程需要保持很久），那么key对应的value就不能被回收了，因为有以下的调用链：Thread->ThreadLocalMap->Entry(key为null)->value**

- **因为value和Thread之间存在这个强引用链路，所以导致value无法回收，就可能出现OOM。**

- **JDK已经考虑到了这个问题，所以在set，remove，resize方法中会扫描kye为null的Entry，并把对应的value设置为null，这样value对象就可以被回收了。**

- ```java
          /**
           * Double the capacity of the table. 源码
           */
          private void resize() {
              Entry[] oldTab = table;
              int oldLen = oldTab.length;
              int newLen = oldLen * 2;
              Entry[] newTab = new Entry[newLen];
              int count = 0;
    
              for (int j = 0; j < oldLen; ++j) {
                  Entry e = oldTab[j];
                  if (e != null) {
                      ThreadLocal<?> k = e.get();
                      if (k == null) {
                          e.value = null; // Help the GC
                      } else {
                          int h = k.threadLocalHashCode & (newLen - 1);
                          while (newTab[h] != null)
                              h = nextIndex(h, newLen);
                          newTab[h] = e;
                          count++;
                      }
                  }
              }
    
              setThreshold(newLen);
              size = count;
              table = newTab;
          }
  ```

- **但是如果一个ThreadLocal不被使用,那么实际上set，remove，rehash方法也不会被调用，如果同时线程又不停止，那么调用链就会一直存在，那么就导致了value的内存泄露。**

- **如何避免内存泄露( 阿里规约)：调用remove方法，就会删除对应的Entry对象，可以避免内存泄露，所以使用完ThreadLocal之后，就应该调用remove方法。**

------

## **NPE问题**

-   

  ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 19:49
   * @description：演示空指针异常
   */
  public class ThreadLocalNPE {
      ThreadLocal<Long> longThreadLocal = new ThreadLocal<>();
  
      public void set(long num) {
          longThreadLocal.set(num);
      }
  
      public long get()//因为个人书写不当的原因 这里会报NPE异常，将long改为Long即可
      {
          return longThreadLocal.get();
      }
  
      public static void main(String[] args) {
          ThreadLocalNPE threadLocalNPE = new ThreadLocalNPE();
          System.out.println(threadLocalNPE.get());
      }
  }
  ```

------

## 补充

- **如果在每个线程中ThreadLocal.set()进去的东西本来就是多个线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身，还是有并发访问问题。**
- **我们在不需要使用ThreadLocal就能解决问题，那么不要强行使用。**
- **优先使用框架的支持，而不是自己创造。**
  - **例如在Spring中，如果可以使用RequestContextHolder，那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove()方法等，造成内存泄露。**
-  **每次HTTP请求都对应一个线程，线程之间相互隔离，这就是ThreadLocal的典型应用场景。**

