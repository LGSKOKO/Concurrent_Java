# 04ThreadLocal源码

## 目录 

- ### **ThreadLocal的关系图**

- ### **ThreadLocal的重要方法**

- ### **ThreadLocal的重要方法的源码**

- ### **ThreadLocalMap类**

- ### **两种使用场景殊途同归**

------

## **ThreadLocal的关系图**

- ![线程池构造函数的参数图](https://raw.github.com/LGSKOKO/Concurrent_Java/master/06ThreadLocal/img/ThreadLocal关系图.jpg)

------

## **ThreadLocal的重要方法**

- **T initialValue()**
  - **该方法会返回当前线程的 初始值，这是一个延迟加载的方法，只有在调用get方法的时候，才会触发。**
  - **当线程第一次使用get方法 访问变量时，将调用此方法，除非线程先调用了set方法，在这种情况下，不会为线程调用initialValue方法。**
  - **通常，每个线程最多调用  一次此方法，但如果已经调用了remove()后，在第偶用get()，则可以再次调用此方法。**
  - **如果不重写本方法，这个方法会返回null。一般使用匿名内部类的方法来重写initialize()方法，以便后续使用中可以初始化副本对象。**
- **void set(T t) ：为这个线程的ThreadLocalMap设置一个新的ThreadLocal键值对。**
- **T get() ：得到这个线程的ThreadLocalMap中ThreadLocal键值对 对应的value。如果是首次调用get()，则会调用initialize来得到这个值。**
- **void remove() ：删除对应这个线程的ThreadLocalMap中的ThreadLocal的键值对。**

------

## ThreadLocal的重要方法的源码

- ```java
  //get方法的源码
  /*get方法是 先取出当前线程的ThreadLocalMap，然后调用map.getEntry方法，把本ThreadLocal的引用作为参数传入，取出map中属于本ThreadLocal的value。
  注意，这个map以及map中的key和value都是保存在线程中的，而不是保存在ThreadLocal中。
  */
  public T get()  {
          Thread t = Thread.currentThread();
          ThreadLocalMap map = getMap(t);
          if (map != null) {
              ThreadLocalMap.Entry e = map.getEntry(this);
              if (e != null) {
                  @SuppressWarnings("unchecked")
                  T result = (T)e.value;
                  return result;
              }
          }
          return setInitialValue();
      }
  ```

- ```java
    /**
       * Get the map associated with a ThreadLocal. Overridden in
       * InheritableThreadLocal.
       *
       * @param  t the current thread
       * @return the map
       */
      ThreadLocalMap getMap(Thread t) {
          return t.threadLocals;
      }
    ```


- ```java
   //set方法的源码
  public void set(T value) {
          Thread t = Thread.currentThread();
          ThreadLocalMap map = getMap(t);
          if (map != null)
              map.set(this, value);
          else
              createMap(t, value);
      }
  ```

- ```java
   //remove方法的源码
  public void remove() {
           ThreadLocalMap m = getMap(Thread.currentThread());
           if (m != null)
               m.remove(this);
       }
  ```
  
- ```java
   private T setInitialValue() {
          T value = initialValue();//设置初始值的时候 需要重写的方法
          Thread t = Thread.currentThread();
          ThreadLocalMap map = getMap(t);
          if (map != null)
              map.set(this, value);
          else
              createMap(t, value);
          return value;
      }
  ```

------

## ThreadLocalMap类 

- **ThreadLocalMap类，也就是Thread.threadLocals ;具体可看源码。**

- **ThreadLocalMap类是每个线程Thread类里面的变量，ThreadLocalMap类里面最重要的是 一个键值对数组Entry[ ] table，可以认为是一个map，键值对：**
- **键：这个ThreadLocal** 
  - **值：实际需要的成员变量，比如user或者simpleDateFormat对象。**
  
- **ThreadLocalMap的hash冲突采用的是线性探测法，也就是如果发生冲突，就继续找一个空位置，而不是用链表拉链。**

- **HashMap的冲突: 分1.8之前和之后两种情况**

****

## **两种使用场景殊途同归**

- **通过源码分析可以看出，setInitialValue和直接set最后都是利用map.set()方法来设置值。**
- **也就是说，最后都会对应到ThreadLocalMap的一个Entry，只不过是起点和入口不一样。**