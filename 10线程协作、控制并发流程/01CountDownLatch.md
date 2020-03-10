# 01CountDownLatch

## 目录

- ### 什么是控制并发流程

- ### CountDownLatch类的作用

- ### CountDownLatch的两个经典用法

- ### CountDownLatch的方法

- ### CountDownLatch的注意事项

- ### 代码示例

------

## 什么是控制并发流程

- **控制并发流程的工具类，作用就是帮助我们程序员更容易得让线程之间合作。**
- **让线程之间相互配合，满足业务逻辑。**

## CountDownLatch类的作用

- **并发流程控制器**
- **倒数门阀（计数器）**
- **例子：购物拼团；游乐园坐过山车排队，人满发车。**
- **流程：倒数结束之前，一直处于等待状态，知道倒计时结束了，此线程才继续工作。**
- **开始——>进入等待——>倒数结束——>继续工作**

## 两个经典用法

- **用法一：一个线程等待多个线程都执行完毕，再继续自己的工作。**
- **用法二：多个线程等待某一个线程的信号，同时开始执行。**

## CountDownLatch的方法

- **CountDownLatch(int count)：只有一个构造函数，参数count为需要倒数的数值。**
- **await():调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行。**
- **countDown():将count值 减1，直到为0时，等待线程会被唤醒。**

## CountDownLatch的注意事项

- **CountDownLatch是不能够重用的，如果需要重新计数，可以考虑用CycliBarrier。**
- **CountDownLatch不能回滚重置。**

## 代码示例

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/19 21:39
   * @description：工厂中，质检，5个工人检查，所有人都认为通过，
   * 产品才合格
   * 对应 用法一
   */
  public class CountDownLatchDemo1 {
      public static void main(String[] args) throws InterruptedException {
          CountDownLatch count = new CountDownLatch(5);
          ExecutorService service = Executors.newFixedThreadPool(5);
          for (int i = 0; i < 5; i++) {
              final int no = i+1;
              Runnable r = new Runnable(){
                  @Override
                  public void run() {
                      try {
                          Thread.sleep((long) (Math.random()*1000));
                          System.out.println("质检员"+no+"检查完毕");
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }finally{
                          count.countDown();
                      }
                  }
              };
              service.submit(r);
          }
  
          System.out.println("等待人员检查产品");
          count.await();
          System.out.println("产品检查完成，通过");
  
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/19 22:04
   * @description：模拟100米赛跑，5名选手都准备好了，只等裁判员的哨声，
   * 所有人开始跑步。
   * 对应 用法二
   */
  public class CountDownLatchDemo2 {
      public static void main(String[] args) throws InterruptedException {
          ExecutorService service = Executors.newFixedThreadPool(5);
          CountDownLatch begin = new CountDownLatch(1);
          for (int i = 0; i < 5; i++) {
              final int no = i+1;
              Runnable runnable = new Runnable(){
  
                  @Override
                  public void run() {
                      try {
                          System.out.println("运动员" + no + "准备好了");
                          begin.await();
                          System.out.println("运动员" + no + "开始飞奔");
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
              };
              service.submit(runnable);
          }
  
          //裁判员检查设备....
          System.out.println("裁判员检查设备....");
          Thread.sleep(5000);
          System.out.println("砰...");
          begin.countDown();
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/19 22:04
   * @description：模拟100米赛跑，5名选手都准备好了，只等裁判员的哨声，
   * 所有人开始跑步；所有人都到达终点，宣布比赛结束。
   * 两种用法的结合
   */
  public class CountDownLatchDemo1And2 {
      public static void main(String[] args) throws InterruptedException {
          ExecutorService service = Executors.newFixedThreadPool(5);
          CountDownLatch begin = new CountDownLatch(1);
          CountDownLatch end = new CountDownLatch(5);
          for (int i = 0; i < 5; i++) {
              final int no = i+1;
              Runnable runnable = new Runnable(){
  
                  @Override
                  public void run() {
                      try {
                          System.out.println("运动员" + no + "准备好了");
                          begin.await();
                          Thread.sleep((long) (Math.random()*1000));
                          System.out.println("运动员" + no + "跑到终点了");
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }finally{
                          end.countDown();
                      }
                  }
              };
              service.submit(runnable);
          }
  
          //裁判员检查设备....
          Thread.sleep(3000);
          System.out.println("砰...,比赛开始");
          begin.countDown();
          end.await();
          System.out.println("所有运动员都到达终点，比赛结束");
      }
  ```

  