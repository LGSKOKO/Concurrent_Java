# 01Future和Callable简介

## 目录

- ### Runnable的缺陷

- ### **Callable接口**

- ### Future接口

-------

## Runnable的缺陷 

- **不能返回一个 返回值。**
- **也不能抛出 checked Exeception。**

## **Callable接口**

- **类似Runnable，被其他线程执行的任务。**
- **有返回值。**
- **不足：  无法在新线程中(new Thread(Runnable r))使用，只能使用ExecutorService ；即 Thread类只支持Runnable。**
- **实现其call方法。**

## Future接口

### Callable和Future的关系

- **我们可以通过Future.get()方法来获取Callable接口返回的执行结果，**
- **可以通过Future.isDone()来判断任务是否已经执行完毕，以及取消这个任务，显示获取任务的结果等。**
- **在call()方法为执行完毕之前，调用get()的线程(假定此时是主线程)会被阻塞，直到call()方法返回结果后，此时future.get()才会的到该结果，然后主线程才会切换到runnable的状态。**
- **所以Future是一个存储器，他存储了call()这个任务的结果，而这个任务的执行时间是无法提前确定的，因为这完全取决call()方法执行的情况。**

### Future的方法

- **get()方法：获取结果。**
  - 1、任务正常完成：get方法会立刻返回结果。
  - 2、任务尚未完成（任务还没开始或者进行中）：get将阻塞并直到任务完成。
  - 3、任务执行过程抛出Exception：不论call()执行时抛出的异常时什么，最后get方法抛出的异常类型都是ExecutionException。
  - 4、任务被取消：get方法会跑出CancellationException。
  - 5、任务超时：get方法会有一个重载方法，是传入一个延迟时间的，如果时间到了还没有获得结果，get方法就会抛出TimeoutException。
- **get(long timeout,TimeUnit unit):有超时的获取。**
  - 如果call()在规定时间内完成了任务，那么就会正常获取到返回值；而如果在指定时间内没有计算出结果，那么就会抛出TimeoutException。
- **cancel()方法：取消任务。**
  - 1、如果这个任务还没有开始执行，那么这种情况最简单，任务会被正常取消，未来也不会执行，方法返回为true。
  - 2、如果任务已完成或已取消，那么cancel方法会执行失败，返回false。
  - 3、如果这个任务已经开始执行了，那么这个取消方法将不会直接取消该任务，而是根据我们填的参数mayInterruptedIfRunning来做判断。true：取消 false：不取消
  - Future.cancel(true)适用于：1、任务能够处理interrupt
  - Future.cancel(false)仅用于避免启动尚未启动的任务，适用于： 1、未能处理interrupt的任务。   2、不清楚任务是否支持取消。  3、需要等待已经开始的任务 执行完成。
- **isDone()方法：判断线程是否执行完毕。**
- **isCanceled()方法：判断是否被取消。**





