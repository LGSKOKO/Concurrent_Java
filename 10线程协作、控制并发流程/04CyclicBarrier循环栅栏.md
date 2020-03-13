# 04CyclicBarrier循环栅栏

## 目录

- ### **CyclicBarrier循环栅栏介绍**

- ### **CyclicBarrier和CountDownLatch的区别**

- ### **代码演示**

------

## **CyclicBarrier循环栅栏介绍**

- **CyclicBarrier循环栅栏和CountDownLatch很类似，都能阻塞一组线程。**

  当有大量线程相互配合 分别计算不同任务，并且需要统一汇总的时候，我们可以使用CyclicBarrier。CyclicBarrier可以构造一个集结点，当某一个线程执行完毕，它就会到集结点等待，直到所有线程都到了集结点，那么该栅栏就被撤销，所有线程再统一出发，继续执行剩下的任务。

- **生活中的例子：“咱们3个人明天中午在学校碰面，都到齐后，一起讨论下学期的计划。”**

## **CyclicBarrier和CountDownLatch的区别**

- **作用不同：CyclicBarrier要固定数量的线程都到了栅栏位置才能继续执行，而CountDownLatch只需等待数字到0，也就是说CountDownLatch用于事件，但是CyclicBarrier用于线程。**
- **可重用性不同：CountDownLatch在倒数到0并触发阀门打开后，就不能再次使用了，除非新建实例；而CyclicBarrier可以重复使用。**

## **代码演示**

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/20 14:29
   * @description：演示CyclicBarrierDemo
   */
  public class CyclicBarrierDemo {
      public static void main(String[] args) {
          CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {
              @Override
              public void run() {
                  System.out.println("一波人都到场了，大家统一出发");
              }
          });
          for (int i = 0; i < 10; i++) {
              new Thread(new Task(i, cyclicBarrier)).start();
          }
      }
  
      static class Task implements Runnable {
          private int id;
          private CyclicBarrier cyclicBarrier;
  
          public Task(int id, CyclicBarrier cyclicBarrier) {
              this.id = id;
              this.cyclicBarrier = cyclicBarrier;
          }
  
          @Override
          public void run() {
              System.out.println("线程" + id + "现在前往集合地点");
              try {
                  Thread.sleep((long) (Math.random() * 1000));
                  System.out.println("线程" + id + "到了集合地点，等待其他人");
                  cyclicBarrier.await();
                  System.out.println("线程" + id + "出发了");
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } catch (BrokenBarrierException e) {
                  e.printStackTrace();
              }
  
          }
      }
  }
  ```

  