# Java 线程创建的 3 种方式
## 1 通过继承 Thread 类
```java
//匿名 Thread 类
new Thread(){
    //重写 run 方法
     @Override
    public void run() {
       //do
    }
}.start();
```

```java
//继承 Thread 类
class MyThread extends Thread {
    @Override
    public void run() {
       //do
    }
}
//假设把 run 方法看成一个任务，那么每创建一个线程，就有一个新的任务
 MyThread myThread = new MyThread();
 myThread.start();
```
- Thread 也实现了 Runnable 接口，需要在 run 方法内捕获处理异常，出现异常会导致线程终止
- 多个线程相互独立，因为创建了多个任务，无法实现资源共享（资源写在 MyThread 类里）
- Java 单继承的特性导致不够灵活



## 2 通过实现 Runnable 接口，然后传入 Thread 启动
```java
//需要借助 Thread 类
new Thread(new Runnable() {
   //重写 run 方法  
     @Override
     public void run() {
       //do
     }
}).start();
```

```java
class MyRunnable implements Runnable {
          @Override
         public void run() {
          //do
         }
}
//其实还是通过 Thread 类对象来创建线程的，每创建一个线程，传入的可以是同一个任务
new Thread(new MyRunnable()).start();
```

- 多个线程协调作业，完成同一个任务，实现了资源共享（资源写在 MyRunnable 类里）
- 可以避免由于 java 单继承特征带来的局限性
- Runnable 可以配合 ExecutorService 线程池的 execute 方法使用



## 3. 通过实现 Callable 接口，借助 FutureTask 类，然后传入 Thread 启动
```java
//同样需要借助 Thread 类
 new Thread(new FutureTask<String>(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return null;
            }
})).start();
```

```java
class MyCallable implements Callable<String>{
     @Override
     public String call() throws Exception {
            return null;
     }
}
MyCallable myCallable = new MyCallable();
//FutureTask 实现了 RunnableFuture<V> 接口，而 RunnableFuture 继承了 Runnable 和 Future<V>
FutureTask<String> futureTask = new FutureTask<>(myCallable);
//Runnable 的特性
new Thread(futureTask).start();
//获取返回值， Future<V> 的特性，注意 get 会阻塞当前线程
String result = futureTask.get();
```
- Callable 类似于 Runnable 接口
- 可以有返回值，可以抛出异常
- 可以配合 ExecutorService 线程池的 submit 方法使用


小结：三种方式最终都是通过 Thread 类对象来真正创建启动线程的


