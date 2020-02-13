# 07线程的未捕获异常UncaughtException应该如何处理

## 目录

- #### **为什么需要UncaughtExceptionHandler？**

- #### **两种解决方案**

- #### **常见面试问题**

------

## 为什么需要UncaughtExceptionHandler？

- **主线程可以轻松发现异常，子线程不行。** 如 错误不能被及时发现，该错误信息在日志文件中可能被忽略等  **例子1**

- **子线程无法用传统的方法捕获。**

- **提高代码的健壮性。**

**例子1：**

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/1/19 20:07
 * @description：单线程，抛出，处理，有异常堆栈，
 * 多线程，子线程发生异常，会有什么不同？
 * @modified By：
 * @version: $
 */
public class ExceptionInChildThread implements Runnable {

    public static void main(String[] args) {
        new Thread(new ExceptionInChildThread()).start();
        for (int i = 0; i < 100; i++) {
            System.out.println(i);
        }
    }
    @Override
    public void run() {
        throw new RuntimeException();
    }
}
```

**例子2：**

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/1/19 20:17
 * @description： 1、不加 try catch抛出4个异常，都打印线程名字
 * 2、加了try cath，期望捕获到第一个线程的异常，线程234不应该运行，希望看到打印出
 * Caught Exception
 *
 * 说明线程的异常不能用传统方法捕获
 * @modified By：
 * @version: $
 */
public class CantCatchDirectly implements Runnable {


    /** 1、
    public static void main(String[] args) throws InterruptedException {
        new Thread(new CantCatchDirectly(),"MyThread-1").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-2").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-3").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-4").start();
    }**/

    /** 2、 这里只所以没有按照上面的意料，是因为 异常发生在子线程，
     * 捕获异常代码实在主线程 所以没办法捕获到子线程的异常**/
    public static void main(String[] args) throws InterruptedException {
        try{
        new Thread(new CantCatchDirectly(),"MyThread-1").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-2").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-3").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-4").start();
    }catch (RuntimeException e){
            System.out.println("Caught Exception");
        }
    }
    @Override
    public void run() {
        throw new RuntimeException();
    }
}
```



------

## 两种解决方案

- **方案一（不推荐）：手动在每个run方法中进行 try catch；太累了 太麻烦了**
- **方案二（推荐）：利用 UncaughtExceptionHandler**
  - **给程序统一设置一个**
  - **给每个线程设置一个**
  - **给线程池设置**

**方案一：**

```java

public class CantCatchDirectly implements Runnable {

    /** 2、 这里只所以没有按照上面的意料，是因为 异常发生在子线程，
     * 捕获异常代码实在主线程 所以没办法捕获到子线程的异常**/
    public static void main(String[] args) throws InterruptedException {
        try{
        new Thread(new CantCatchDirectly(),"MyThread-1").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-2").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-3").start();
        Thread.sleep(300);
        new Thread(new CantCatchDirectly(),"MyThread-4").start();
    }catch (RuntimeException e){
            System.out.println("Caught Exception");
        }
    }
    @Override
    public void run() {
        try { //新添加代码
            throw new RuntimeException();
        } catch (RuntimeException e) {
            System.out.println("Caught Exception");
        }
    }
}
```

**方案二：**

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/1/20 9:07
 * @description：使用刚刚编写的UncaughtExceptionHandler
 * @modified By：
 * @version: $
 */
public class UseOwnUncaughtExceptionHandler implements Runnable {

    public static void main(String[] args) throws InterruptedException {
        Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler("捕获器1"));
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread1").start();
        Thread.sleep(300);
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread2").start();
        Thread.sleep(300);
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread3").start();
        Thread.sleep(300);
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread4").start();


    }
    @Override
    public void run() {
        throw new RuntimeException();
    }
}
```

```java
/**
 * @author ：李先生
 * @date ：Created in 2020/1/20 8:54
 * @description：自己的UncaughtExceptionHandler
 * @modified By：
 * @version: $
 */
public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
    private String name;
    public MyUncaughtExceptionHandler(String name)
    {
        this.name = name;
    }
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        Logger logger = Logger.getAnonymousLogger();
        logger.log(Level.WARNING,"线程异常，终止了"+t.getName(),e);
        System.out.println(name+"捕获了异常"+t.getName()+"异常"+e);
    }
}
```

------

## **常见面试问题**

- Java**异常体系**
- 如何**全局处理**异常？为什么要全局处理？不处理行不行？
- run方法**是否可以跑出异常**？如果抛出异常，**线程的状态**会怎么样?
- 线程中如何处理某个**未处理异常**？