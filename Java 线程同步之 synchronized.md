# Java 线程同步之 synchronized

## synchronized 关键字
- 是一种重量级锁，是 JVM 提供的同步原语之一
- 通过 Monitor 对象监视器锁机制（内置锁）实现线程同步操作，通过 monitorenter 和 monitorexit 字节码指令（底层实现）获取和释放锁（是原子操作，不可中断，确保临界区的互斥性）
- 分为对象锁和类锁

## 线程安全
- 单线程不存在线程安全问题，而多个线程可能会存在同时访问同一个资源（变量、对象、文件、数据库，这种资源又叫临界资源、共享资源）的情况，会导致程序运行结果不是预期结果

## 问题示例
```java
public class MyRunnable implements Runnable {
    private static int count; //注意 static
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        for (int i = 0; i < 10000; i++) {
            count++;
        }
        //打印 count 结果不是预期值
        System.out.println(this + " " + Thread.currentThread() + " count " + +count);
    }
}
//
public static void main(String[] args) {
    MyRunnable myRunnable = new MyRunnable();
    //两个线程用的是同一个 MyRunnable 实例对象
    new Thread(myRunnable).start();
    new Thread(myRunnable).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@54998dc8 Thread[Thread-1,5,main] count 15621
//com.louisgeek.MyRunnable@54998dc8 Thread[Thread-0,5,main] count 16073
```

## 解决方案
- 同步互斥访问
- 在同一时刻，确保最多只能有一个线程访问该临界资源，当一个线程在访问临界资源的代码时上加一个锁，当用完临界资源后释放锁，从而让其他线程能够继续访问该资源，可以通过 synchronized 或 Lock 来实现同步互斥访问

## 对象锁
1. 加入 synchronized(this) 代码块锁（对象锁）
```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        //此时的 this 就是 MyRunnable.this
        synchronized(this) {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
            System.out.println(this + " " + Thread.currentThread() + " count " + +count);
        }
    }
}
//
public static void main(String[] args) {
    MyRunnable myRunnable = new MyRunnable();
    //两个线程用的是同一个 MyRunnable 实例对象
    new Thread(myRunnable).start();
    new Thread(myRunnable).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@44f12c70 Thread[Thread-0,5,main] count 10000
//com.louisgeek.MyRunnable@44f12c70 Thread[Thread-1,5,main] count 20000
```

2. 也可以加入 synchronized 方法锁（对象锁）
```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    //方法锁默认的锁对象是 this，也就是 MyRunnable.this
    public synchronized void methodTest() {
        for (int i = 0; i < 10000; i++) {
            count++;
        }
        System.out.println(this + " " + Thread.currentThread() + " count " + +count);
    }
}
//
public static void main(String[] args) {
    MyRunnable myRunnable = new MyRunnable();
    //两个线程用的是同一个 MyRunnable 实例对象
    new Thread(myRunnable).start();
    new Thread(myRunnable).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@406fd94d Thread[Thread-0,5,main] count 10000
//com.louisgeek.MyRunnable@406fd94d Thread[Thread-1,5,main] count 20000
```

## 类锁
- 如果修改成使用多个 MyRunnable 实例对象
```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        //此时的 this 就是 MyRunnable.this
        synchronized(this) {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
            //此时打印 count 结果又不是预期值了
            System.out.println(this + " " + Thread.currentThread() + " count " + +count);
        }
    }
}
//
public static void main(String[] args) {
    MyRunnable myRunnable = new MyRunnable();
    MyRunnable myRunnable2 = new MyRunnable();
    new Thread(myRunnable).start();
    new Thread(myRunnable2).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@141cded9 Thread[Thread-1,5,main] count 12370
//com.louisgeek.MyRunnable@519868d6 Thread[Thread-0,5,main] count 14802
```

1. 修改加入 synchronized(MyRunnable.class) 代码块锁（类锁）
```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        synchronized (MyRunnable.class) {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
            System.out.println(this + " " + Thread.currentThread() + " count " + +count);
        }
    }
}
//
public static void main(String[] args) {
    MyRunnable myRunnable = new MyRunnable();
    MyRunnable myRunnable2 = new MyRunnable();
    new Thread(myRunnable).start();
    new Thread(myRunnable2).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@2034c894 Thread[Thread-0,5,main] count 10000
//com.louisgeek.MyRunnable@2cf062b0 Thread[Thread-1,5,main] count 20000
```


2. 也可以加入 synchronized 静态方法锁（类锁）
```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }

    public static synchronized void methodTest() {
    for (int i = 0; i < 10000; i++) {
                count++;
     }
    System.out.println(this + " " + Thread.currentThread() + " count " + +count);
 }
}
//
public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        MyRunnable myRunnable2 = new MyRunnable();
        new Thread(myRunnable).start();
        new Thread(myRunnable2).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@5024c814 Thread[Thread-1,5,main] count 10000
//com.louisgeek.MyRunnable@30f16220 Thread[Thread-0,5,main] count 20000
```










