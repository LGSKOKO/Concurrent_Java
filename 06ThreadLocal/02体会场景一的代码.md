## **02通过SimpleDateFomat进化之路  体会场景一**

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 11:14
   * @description：演示用两个线程打印日期
   */
  public class ThreadLocalNormalUsage00 {
      public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  String date = new ThreadLocalNormalUsage00().date(10);
                  System.out.println(date);
              }
          }).start();
          new Thread(new Runnable() {
              @Override
              public void run() {
                  String date = new ThreadLocalNormalUsage00().date(1071);
                  System.out.println(date);
              }
          }).start();
      }
  
      public String date(int seconds){
          //参数的单位是毫秒 从1970.1.1 00:00:00开始
          //这里 我们之所以会显示 1970-01-01 08:xx:xx
          // 是因为我们是东八区
          Date date = new Date(1000*seconds);
          SimpleDateFormat dateFormat =
                  new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
          return dateFormat.format(date);
      }
  }
  
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 11:14
   * @description：演示用10个线程打印日期 可用for循环实现
   * 但是如果我们需求是1000个打印日期的任务呢，那我们依然可以for来创建线程，
   * 但是这样会消耗内存太多，所以我们必然要用线程池
   */
  public class ThreadLocalNormalUsage01 {
  
      public static void main(String[] args) {
          for (int i = 0; i < 10; i++) {
              final int finalI = i;
              new Thread(new Runnable() {
                  @Override
                  public void run() {
                      String date = new ThreadLocalNormalUsage01().date(finalI * 10);
                      System.out.println(date);
                  }
              }).start();
              try {
                  Thread.sleep(500);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
  
      }
  
      public String date(int seconds) {
          //参数的单位是毫秒 从1970.1.1 00:00:00开始
          //这里 我们之所以会显示 1970-01-01 08:xx:xx
          // 是因为我们是东八区
          Date date = new Date(1000 * seconds);
          SimpleDateFormat dateFormat =
                  new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
          return dateFormat.format(date);
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 11:14
   * @description：演示用1000个线程打印日期
   * 这里我们虽然解决了上一个问题 创建太多线程带来内存开销问题，
   * 但是我们仔细阅读代码 会发现 这里会创建1000个SimpleDateFormat对象
   * 因重复创建对象也是一笔很大的开销，所以我们对此还要进行优化
   */
  public class ThreadLocalNormalUsage02 {
  
      public static ExecutorService threadPool =
              Executors.newFixedThreadPool(10);
  
      public static void main(String[] args) {
          for (int i = 0; i < 1000; i++) {
              final int finalI = i;
              threadPool.submit(new Runnable() {
                  @Override
                  public void run() {
                      String date = new ThreadLocalNormalUsage02().date(finalI*10);
                      System.out.println(date);
                  }
              });
          }
          threadPool.shutdown(); //关闭线程池
      }
  
      public String date(int seconds){
          //参数的单位是毫秒 从1970.1.1 00:00:00开始
          //这里 我们之所以会显示 1970-01-01 08:xx:xx
          // 是因为我们是东八区
          Date date = new Date(1000*seconds);
          SimpleDateFormat dateFormat =
                  new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
          return dateFormat.format(date);
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 11:14
   * @description：演示用1000个线程打印日期 并使用static变量
   * 来避免重复创建对象带来的开销 ，但是我们仔细观察输出 会发现有的输出时间一样
   * 说明我们这里存在线程安全问题
   */
  public class ThreadLocalNormalUsage03 {
  
      public static ExecutorService threadPool =
              Executors.newFixedThreadPool(10);
      static SimpleDateFormat dateFormat =
              new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
  
      public static void main(String[] args) {
          for (int i = 0; i < 1000; i++) {
              final int finalI = i;
              threadPool.submit(new Runnable() {
                  @Override
                  public void run() {
                      String date = new ThreadLocalNormalUsage03().date(finalI * 10);
                      System.out.println(date);
                  }
              });
          }
          threadPool.shutdown(); //关闭线程池
      }
  
      public String date(int seconds) {
          //参数的单位是毫秒 从1970.1.1 00:00:00开始
          //这里 我们之所以会显示 1970-01-01 08:xx:xx
          // 是因为我们是东八区
          Date date = new Date(1000 * seconds);
          return dateFormat.format(date);
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 11:14
   * @description：演示用1000个线程打印日期 并使用static变量 并进行加锁
   * 这里加锁虽然可以避免上一个演示存在的线程安全问题，但是我们知道加锁
   * 只能让同一时刻 只能有一个线程运行，但是这样会使我们的效率降低即性能问题
   * 所以我们将引入更好的解决方案 ThreadLocal
   */
  public class ThreadLocalNormalUsage04 {
  
      public static ExecutorService threadPool =
              Executors.newFixedThreadPool(10);
      static SimpleDateFormat dateFormat =
              new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
  
      public static void main(String[] args) {
          for (int i = 0; i < 1000; i++) {
              final int finalI = i;
              threadPool.submit(new Runnable() {
                  @Override
                  public void run() {
                      String date = new ThreadLocalNormalUsage04().date(finalI * 10);
                      System.out.println(date);
                  }
              });
          }
          threadPool.shutdown(); //关闭线程池
      }
  
      public String date(int seconds) {
          //参数的单位是毫秒 从1970.1.1 00:00:00开始
          //这里 我们之所以会显示 1970-01-01 08:xx:xx
          // 是因为我们是东八区
          Date date = new Date(1000 * seconds);
          String s = null;
          //进行加锁
          synchronized (ThreadLocalNormalUsage04.class) {
              s = dateFormat.format(date);
          }
          return s;
      }
  }
  ```

- ```java
  /**
   * @author ：李先生
   * @date ：Created in 2020/2/10 11:14
   * @description：演示用1000个线程打印日期 利用ThreadLocal，给每个线程分配自己的dateFormat对象，
   * 保证了线程安全，并且能高效利用内存
   * 这里是因为线程池每个工作线程内 都有自己独享的simpleDateFormat对象
   */
  public class ThreadLocalNormalUsage05 {
  
      public static ExecutorService threadPool =
              Executors.newFixedThreadPool(10);
  
      public static void main(String[] args) {
          for (int i = 0; i < 1000; i++) {
              final int finalI = i;
              threadPool.submit(new Runnable() {
                  @Override
                  public void run() {
                      String date = new ThreadLocalNormalUsage05().date(finalI * 10);
                      System.out.println(date);
                  }
              });
          }
          threadPool.shutdown(); //关闭线程池
      }
  
      public String date(int seconds) {
          //参数的单位是毫秒 从1970.1.1 00:00:00开始
          //这里 我们之所以会显示 1970-01-01 08:xx:xx
          // 是因为我们是东八区
          Date date = new Date(1000 * seconds);
          SimpleDateFormat dateFormat =
                  ThreadSafeFormatter.dateFormatThreadLocal.get();
          return dateFormat.format(date);
      }
  }
  
  class ThreadSafeFormatter {
      //    public static ThreadLocal<SimpleDateFormat>
  //    dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>(){
  //        @Override
  //        protected SimpleDateFormat initialValue() {
  //            return new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
  //        }
  //    };
      //下面是 lambda表达式的写法
      public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal
              = ThreadLocal.withInitial(
              () -> new SimpleDateFormat("yyy-MM-dd hh:mm:ss"));
  }
  ```

  

