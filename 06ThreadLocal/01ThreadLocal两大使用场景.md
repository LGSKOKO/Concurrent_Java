# 01ThreadLocal两大使用场景

## 目录

- ### **两大使用场景 简介**

- ### **通过SimpleDateFomat进化之路  体会场景一**

- ### **通过实例  体会场景二**

- ### **ThreadLocal的两个作用**

- ### **ThreadLocal使用的注意点**

------

## **两大使用场景 简介**

- **场景一：每个线程需要一个独享的对象 (通常是 工具类)，典型需要使用的类有SimpleDateFormate和Random），主要是因为工具类存在 线程不安全问题。**

  - **比喻：教材书只有一本，大家一起在这本教材书上做笔记时 会存在线程安全问题，复印教材书后，每人一本 复印的教材书，当大家一起做笔记时 不存在线程安全问题**

- **场景二：每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦。**

  - **强调的是同一个请求（同一个线程内）不同方法间的共享。**


------

## **通过SimpleDateFomat进化之路  体会场景一**

-  **[代码演示](https://github.com/LGSKOKO/Concurrent_Java/blob/master/06ThreadLocal/02体会场景一的代码.md)** 

------

## **通过实例  体会场景二**

- **实例：当前用户信息需要被线程内所有方法共享，即信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的**

- **一个比较繁琐的解决方案是  把user作为参数层层传递，从service-1()到service-2()，再从service-2()到service-3()，以此类推，但是这样会导致 代码冗余且不易维护。  **

- **我们可以用全局变量解决，但是当多线程同时工作时，我们需要保证线程安全的，可以用synchronized也可以用ConcurrentHashMap，但无论用什么，都会对性能有所影响。**

- **更好的办法是使用 ThreadLocal，这样无需synchronized，可以在不影响性能的情况下，也无需层层层传递参数，就可达到保存当前线程对应的用户信息的目的。**

  

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 15:07
   * @description： 演示ThreadLocal用法2：
   * 避免传递参数的麻烦,也不影响性能
   * 强调的是同一个请求（同一个线程内）不同方法间的共享。
   * 上一个场景是 因为需要初始值，所以要么重写initialValue方法或调用类的withInitialValue方法。。
   * 这里不需要初始值，所以不需要重写initialValue()方法，但是必须手动调用set()方法。
   */
  public class ThreadLocalNormalUsage06 { 
      public static void main(String[] args) {
          new Service1().process();
      }
  }
  
  class Service1 {
      public void process() {
          User user = new User("kk");
          UserContextHolder.holder.set(user);
          new Service2().process();
      }
  }
  
  class Service2 {
      public void process() {
          User user = UserContextHolder.holder.get();
          System.out.println("Service3拿到用户名：" + user.name);
          new Service3().process();
      }
  }
  
  class Service3 {
      public void process() {
          User user = UserContextHolder.holder.get();
          System.out.println("Service3拿到用户名：" + user.name);
      }
  }
  
  class UserContextHolder {
      //这里是第二种用法 注意与第一种用法 的区别
      public static ThreadLocal<User> holder = new ThreadLocal<>();
  
  }
  
  class User {
      String name;
  
      public User(String name) {
          this.name = name;
      }
  } 
  
  ```

------

## **ThreadLocal的两个作用**

- **让某个需要用到的对象在线程间隔离（每个线程都有自己的 独立的对象）。**
- **在线程的任何方法中都可以轻松获取到 该对象。**
- **可通过回顾 上述的两个场景，对应这两个作用。**

------

## **ThreadLocal使用的注意点**

- **我们应该根据共享对象的生成时机不同，选择initialValue或set来保存对象。**
- **场景一：initialValue**
  - **在ThreadLocal第一次get的时候把对象初始化出来， 对象的初始化时机可以 由我们控制。**
- **场景二：set**
  - **如果需要保存到ThreadLocal里的 对象的生成时机 不由我们随意控制时；例如拦截器生成的用户信息。**