# 02Future的使用

## 目录

- ### 用法一

- ### 用法二

- ### Future的注意点

------

## 用法一

- **用法一：线程池的submit方法返回Future对象**

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/22 21:24
   * @description：演示一个Future
   */
  public class OneFuture {
  
      public static void main(String[] args) {
          ExecutorService service = Executors.newFixedThreadPool(10);
          Future<Integer> future = service.submit(new CallableTask());
          try {
              System.out.println(future.get());
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (ExecutionException e) {
              e.printStackTrace();
          }
          service.shutdown();
      }
      static class CallableTask implements Callable<Integer>{
          @Override
          public Integer call() throws Exception {
              Thread.sleep(3000);
              return new Random().nextInt();
          }
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/22 21:31
   * @description：演示一个future，lambda形式
   */
  public class OneFutureLambda {
      public static void main(String[] args) {
          ExecutorService service = Executors.newFixedThreadPool(10);
          Callable<Integer> callable = ()->{
              Thread.sleep(3000);
              return new Random().nextInt();
          };
          Future<Integer> future = service.submit(callable);
          service.shutdown();
          try {
              System.out.println(future.get());
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (ExecutionException e) {
              e.printStackTrace();
          }
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/22 21:41
   * @description：演示批量提交任务，用List来批量接收结果。
   */
  public class MultiFuture {
  
      public static void main(String[] args) {
          ExecutorService service = Executors.newFixedThreadPool(5);
          ArrayList<Future> futures = new ArrayList<>();
          for (int i = 0; i < 20; i++) {
              Future<Integer> future = service.submit(new CallableTask());
              futures.add(future);
          }
          service.shutdown();
          for (int i = 0; i < 20; i++) {
              Future<Integer> future = futures.get(i);
              try {
                  Integer integer = future.get();
                  System.out.println(integer);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } catch (ExecutionException e) {
                  e.printStackTrace();
              }
          }
      }
  
      static class CallableTask implements Callable<Integer> {
          @Override
          public Integer call() throws Exception {
              Thread.sleep(3000);
              return new Random().nextInt();
          }
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/22 21:53
   * @description： 演示get方法过程中抛出异常，for循环为了演示Exception的时机，
   * 并不是说一产生异常就抛出，直到我们get执行时，才会抛出
   */
  public class GetException {
      public static void main(String[] args) {
          ExecutorService service = Executors.newFixedThreadPool(10);
          Future<Integer> future = service.submit(new CallableTask());
          try {
              for (int i = 0; i < 5; i++) {
                  System.out.println(i);
                  Thread.sleep(200);
              }
              System.out.println(future.isDone());//只管任务是否完成了 不管是成功还是失败
             future.get();//只有get的时候才会抛出异常
          } catch (InterruptedException e) {
              e.printStackTrace();
              System.out.println("InterruptedException");
          } catch (ExecutionException e) {
              e.printStackTrace();
              System.out.println("ExecutionException");
          }
          service.shutdown();
      }
      static class CallableTask implements Callable<Integer> {
          @Override
          public Integer call() throws Exception {
            throw new IllegalArgumentException("Callable抛出异常");
          }
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/22 22:42
   * @description： 演示get的超时方法，需要注意超时后处理，调用future.cancel（）。
   * 演示cancel传入true和false的区别，代表是否中断正在执行的任务。
   */
  public class Timeout {
  
      private static final Ad DEFAULT_AD = new Ad("无网络时候的默认广告");
      private static final ExecutorService exec = Executors.newFixedThreadPool(5);
      static class Ad{
          String name;
          public Ad(String name) {
              this.name = name;
          }
          @Override
          public String toString() {
              return "Ad{" +
                      "name='" + name + '\'' +
                      '}';
          }
      }
  
      static class FetchAdTask implements Callable<Ad>{
  
          @Override
          public Ad call() throws Exception {
              try{
                  Thread.sleep(3000);
              }catch (InterruptedException e){
                  System.out.println("sleep期间被中断了");
                  return new Ad("被中断时候的默认广告");
              }
              return new Ad("订票广告");
          }
      }
  
      public void printAd(){
          Future<Ad> f = exec.submit(new FetchAdTask());
          Ad ad = null;
          try {
              ad = f.get(2000,TimeUnit.MILLISECONDS);
          } catch (InterruptedException e) {
              ad = new Ad("被中断时候的默认广告");
          } catch (ExecutionException e) {
              ad = new Ad("异常时候的默认广告");
          } catch (TimeoutException e) {
              ad = new Ad("超时时候的默认广告");
              System.out.println("超时，为获取到广告");
  //            boolean cancel = f.cancel(false);
              boolean cancel = f.cancel(false);
              System.out.println("cancel的结果"+cancel);
          }
          exec.shutdown();
          System.out.println(ad);
      }
  
      public static void main(String[] args) {
          Timeout timeout = new Timeout();
          timeout.printAd();
      }
  }
  ```

------

## 用法二

- **用FutureTask来创建Future**
- 图片

- **所以它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。**

  把Callable实例当做参数，生成FutureTask的对象，然后把这个对象当做一个Runnable对象，用线程池或另外线程去执行这个Runnable对象，最后通过FutureTask获取刚才执行的结果。

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/23 9:04
   * @description：演示FutureTask的用法
   */
  public class FutureTaskDemo {
  
      public static void main(String[] args) {
          Task task = new Task();
          FutureTask<Integer> futureTask = new FutureTask(task);
  //        new Thread(futureTask).start();		//这是放在线程中去执行
          ExecutorService service = Executors.newCachedThreadPool();	//这是放在线程池中执行
          service.submit(futureTask);
          try {
              System.out.println("计算的结果："+futureTask.get());
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (ExecutionException e) {
              e.printStackTrace();
          }
  
      }
      
      static class Task implements Callable<Integer>{
          @Override
          public Integer call() throws Exception {
              int sum=0;
              for (int i = 0; i < 100; i++) {
                  sum+=i;
              }
              return sum;
          }
      }
  }
  ```

  