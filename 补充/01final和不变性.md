# 01final和不变性

## 目录

- ### **什么是不变性（Immutable）**

- ### **final的作用**

- ### **final的3种用法：修饰变量、方法、类**

- ### **不变性和final的关系**

------

## **什么是不变性（Immutable）**

- **如果对象再被创建后，状态就不能被修改，那么它就是不可变的。**
- **具有不可变的对象一定是线程安全的，我们不需要对其采取任何额外的安全措施，也能保证线程安全。**

------

## final的作用

- **类防止被继承、方法防止被重写、变量防止被修改。**
- **天生是线程安全的，而且不需要额外的同步开销。**

------

## final的3种用法：修饰变量、方法、类

- **final修饰 变量，意味着值不能被修改。如果变量是对象，那么对象的引用不能变，但是对象自身的内容依然是可以变化的。**
  - **final instance variable （类中的final属性）**
    - **第一种是 在声明变量的 等号右边 直接赋值。**
    - **第二种是 构造函数中 赋值。**
    - **第三种是 类的初始化代码块中赋值（不常用）。**
  - **final static variable （类中的static final属性）**
    - **第一种是 在声明变量的 等号右边 直接赋值。**
    - **第二种是 类的静态代码块中赋值。**
  - **final local variable （方法中的final变量）**
    - **没有具体规定赋值时期，但是必须在使用前 进行赋值。**
- **final修饰方法：**
  - **构造方法不能被final修饰 。**
  - **被final修饰的方法 不可被重写。**
  - **引申：static方法不能被重写。**
- **final修饰 类：类不可被继承。**

-------

## **不变性和final的关系**

- **不变性并不意味着，简单地用final修饰就是不可变：**

  - **对于基本数据类型，确实被final修饰后就具有不可变性。**
  - **但是对于对象类型，需要该对象保证自身被创建后，状态永远不会变 才可以。**

- **满足以下条件时，对象才是不可变的：**

  - **对象创建后，其状态就不能被修改。**
  - **所有的属性都是final修饰的。**
  - **对象创建过程中没有发生逸出。**

- **把变量写在线程内部——栈封闭：**

  - **在方法里新建的局部变量，实际上是存储在每个线程私有的栈空间，而不是每个栈的占空间是不能被其他线程所访问到的，所以不会有线程安全问题。这就是著名的“栈封闭”技术。**

  - ```java
    /**
     * 描述：     演示栈封闭的两种情况，基本变量和对象 先演示线程争抢带来错误结果，然后把变量放到方法内，情况就变了
     */
    public class StackConfinement implements Runnable {
    
        int index = 0;
    
        public void inThread() {
            int neverGoOut = 0;
            synchronized (this) {
                for (int i = 0; i < 10000; i++) {
                    neverGoOut++;
                }
            }
    
            System.out.println("栈内保护的数字是线程安全的：" + neverGoOut);
        }
    
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                index++;
            }
            inThread();
        }
    
        public static void main(String[] args) throws InterruptedException {
            StackConfinement r1 = new StackConfinement();
            Thread thread1 = new Thread(r1);
            Thread thread2 = new Thread(r1);
            thread1.start();
            thread2.start();
            thread1.join();
            thread2.join();
            System.out.println(r1.index);
        }
    }
    ```

    